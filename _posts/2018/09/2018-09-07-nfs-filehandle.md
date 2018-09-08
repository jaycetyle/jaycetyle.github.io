---
layout: post
title: Linux NFS filehandle 筆記
description: 最近因為在看 Linux NFS 相關的程式碼，趁還有點記憶的時候來筆記一下，這篇主要筆記 NFS filehandle，是 NFS 操作遠端檔案系統的最基礎機制之一，用來表示一個文件。
modified: 2018-09-08
tags: [大魔導士]
categories: [作業系統, Linux Kernel]
image:
    feature: 
    credit: 
    creditlink: 
---

　　最近在看 Linux NFS 相關的程式碼，趁還有點記憶的時候來筆記一下，這篇主要筆記 NFS filehandle 的運作機制。

<!--more-->

### 基本概念
　　filehandle 是 NFS 操作遠端檔案系統的最基礎機制之一，用來表示一個對應的文件，從 C 語言的角度來看，他很類似 fopen 所得到的 file descriptor，對於使用者而言，filehandle/file descriptor 是一種不透明(opaque)的資料結構，使用者不需要知道他實際上存了些什麼，只要知道有他就可以操作到對應的檔案。

　　相較於直接使用檔案路徑來表示一個檔案，使用 filehandle 有兩個主要的優點及目的，首先，filehandle 通常是一個較小且較為固定的大小，對網路傳輸有其便利性，其次是他總是代表相同的檔案，就算對象檔案被重新命名或是被移動過，他還是能找回原本的那個檔案。

### filehandle 結構
　　filehandle 主要包含了兩個主要成份，分別是代表匯出路徑(export point)檔案系統的 fsid，以及代表檔案系統內一個檔案的 fileid，我們可以看 Linux 核心內是如何定義他：

{% highlight c %}
struct nfs_fhbase_new {
	__u8		fb_version;	/* 代表版本，目前固定為 1, even => nfs_fhbase_old */
	__u8		fb_auth_type;	/* deprecated 沒有被使用 */
	__u8		fb_fsid_type;
	__u8		fb_fileid_type;
	__u32		fb_auth[1];	/* deprecated 沒有被使用 */
/*	__u32		fb_fsid[0]; floating */
/*	__u32		fb_fileid[0]; floating */
};
{% endhighlight %}

　　fb_auth_type 及 fb_auth 已經是 deprecated 沒有被使用，因此主要的部份就只剩下 fsid 及 fileid。

### fsid
　　fb_fsid_type 用來表示後面的 fb_fsid 如何表示一個匯出的檔案系統，這兩個值我沒完全搞懂他到底是如何在 user space 和 kernel space 之間傳遞，之後要找時間再深入看一下，但就我目前認知的是，他是由 nfs-utils 內的 **mountd** 來做初始設定。

　　剛開始我也很疑惑為何是 mountd，但仔細思考了一下，給掌管匯出路徑的 mountd 來指定 fsid 也是合理的。也許會問，平常更新 /etc/exports 時都是透過 exportfs 來重新載入，難道不是 exportfs 在管理匯出嗎？但實際上 exportfs 比較像是 mountd 的輔助程式，真正和核心溝通的還是 mountd。

　　mountd 在決定 fsid 時大致上有二種方式，第一種是使用者在 /etc/exports 內直接指定 fsid=NUM，第二種是使用 UUID，而 UUID 又有兩種取得方式，第一種，也是優先選擇的是檔案系統掛載的 blkid，第二種是透過 statfs 取得 filesystem 的 UUID。

　　不同的 filesystem 適用於不同的 UUID 取得方式，例如 FAT 透過 statfs 得到的 fsid 是 device number，但 device number 是可能會改變的，因此並不恰當。相反的例子是 btrfs 的 subvolume，每個 subvolume 有不同的 fsid，但共用同一個 blkid，btrfs 使用 fsid 會有他的意義。

　　相關的程式碼位於 [nfs-utils/utils/mountd/cache.c](http://git.linux-nfs.org/?p=steved/nfs-utils.git;a=blob;f=utils/mountd/cache.c) 內的 uuid_by_patch 函式。

### fileid
　　fileid 的編碼(encode) 則是由每個檔案系統自行實作，這也是一個檔案系統要支援 NFS 最基本需要實作的部份，相關的函式介面定義在以下結構，最核心的函數為 encode_fh、fh_to_dentry、fh_to_parent 及 get_parent：

{% highlight c %}
struct export_operations {
	int (*encode_fh)(struct inode *inode, __u32 *fh, int *max_len,
			struct inode *parent);
	struct dentry * (*fh_to_dentry)(struct super_block *sb, struct fid *fid,
			int fh_len, int fh_type);
	struct dentry * (*fh_to_parent)(struct super_block *sb, struct fid *fid,
			int fh_len, int fh_type);
	int (*get_name)(struct dentry *parent, char *name,
			struct dentry *child);
	struct dentry * (*get_parent)(struct dentry *child);

	/* 在這以下的函數只有 xfs 有實作 */
	int (*commit_metadata)(struct inode *inode);

	/* 以下幾個函數和 pnfs 有關，但我還不確定用途 */
	int (*get_uuid)(struct super_block *sb, u8 *buf, u32 *len, u64 *offset);
	int (*map_blocks)(struct inode *inode, loff_t offset,
			  u64 len, struct iomap *iomap,
			  bool write, u32 *device_generation);
	int (*commit_blocks)(struct inode *inode, struct iomap *iomaps,
			     int nr_iomaps, struct iattr *iattr);
};
{% endhighlight %}

　　encode_fh 將一個檔案的 inode 轉為 fileid，如果 parent 不為 NULL 的話，必須將該檔案的 parent 目錄 inode 也一並存入 fileid。這個函數有預設實作，會存 32bits inode number 及 inode generation，如以下結構：

{% highlight c %}
struct {
	u32 ino;
	u32 gen;
	u32 parent_ino;
	u32 parent_gen;
} i32;
{% endhighlight %}

　　我們可以在核心的 fs/exportfs/expfs.c 找到這個預設的實作：

{% highlight c %}
static int export_encode_fh(struct inode *inode, struct fid *fid,
		int *max_len, struct inode *parent)
{
	int len = *max_len;
	int type = FILEID_INO32_GEN;

	// 確認提供的空間夠大，不夠大的話則將所需要的大小透過 max_len 回傳給呼叫者
	if (parent && (len < 4)) {
		*max_len = 4;
		return FILEID_INVALID; // 0xFF 代表無效的 FILEID
	} else if (len < 2) {
		*max_len = 2;
		return FILEID_INVALID;
	}

	// 填入檔案的 inode number 及 generation
	len = 2;
	fid->i32.ino = inode->i_ino;
	fid->i32.gen = inode->i_generation;

	// 如果呼叫者有提供 parent，則將 parent 一併編入 fileid
	if (parent) {
		fid->i32.parent_ino = parent->i_ino;
		fid->i32.parent_gen = parent->i_generation;
		len = 4;
		type = FILEID_INO32_GEN_PARENT;
	}
	*max_len = len;	// 將實際使用的大小從 max_len 回傳給呼叫者
	return type;	// 回傳編碼的方式
}
{% endhighlight %}

　　每個檔案系統根據其需求可自行編碼不同的內容及大小，只要能確保找到唯一對應的檔案即可，例如 btrfs 至少得多存 root object ID，因為他的 inode number 在不同 subvolume 內是可能重複的。

　　fh_to_dentry 實作 fileid 的解碼工作，給定一個 fileid，回傳檔案的 dentry，這個就沒有預設實作了，每個檔案系統要根據他的 layout 方式取回對應的 dentry，之後 nfs server 便可以透過該 dentry 進行檔案操作。

　　fh_to_parent 實作 fileid 的另一部份解碼工作，如果 parent 資訊有存入 fileid，則回傳該 fileid 內所紀錄的 parent dentry。get_parent 則是給定一個 dentry，回傳該他的 parent dentry。

　　需要 parent 資訊的主要原因是 subtree_check，當 NFS 伺服端啟用 subtree_check 後，進行檔案操作前會先經過權限的檢查，要進行權限檢查，就勢必要能取回該檔案的完整路徑。NFS 取回完整路徑的方式是透過 fh_to_parent 及 get_parent 這兩個函數，先利用 fh_to_parent 取得檔案的 parent dentry，再透過 get_parent 一路找回根目錄。

　　已經有 get_parent 函數了為何還要 fh_to_parent 呢？熟悉 ext4 檔案系統架構者就會知道，只有一個檔案的 inode number 是沒辦法直接取得他所在的目錄的，所以才需要在編碼時將他的 parent 一並編入。但如果已知一個目錄，檔案系統就可以透過 '..' 一路找回根目錄了。