---
layout: post
title: Linux 系統程式設計 - fd 及 open()、close() 系統呼叫
description: "開始接觸 Linux Kernel 也有差不多一年的時間，最近開始有明顯地感覺到有某種瓶頸存在，仔細思考了一下覺得是底子不夠，所以決定從基礎來好好學習一下，再搭配核心程式碼來確認是否是看到的那樣。這篇主要筆記 file descriptor、open() 及 close() 系統呼叫相關的部。"
modified: 2018-12-30
tags: [小法師, 巫師]
categories: [C/C++, Linux Kernel]
image:
    feature: 
    credit: 
    creditlink: 
---

　　開始接觸 Linux Kernel 也有差不多一年的時間，最近開始有明顯地感覺到有某種瓶頸存在，仔細思考了一下覺得是底子不夠，所以決定從基礎來好好學習一下，再搭配核心程式碼來確認是否是看到的那樣。這篇主要筆記 file descriptor、open() 及 close() 系統呼叫相關的部分，主要參考 [Robert Love 的 Linux System Programming](https://www.tenlong.com.tw/products/9789862769812)。

<!--more-->　

## File Descriptor

　　檔案必須先開啟後才能進行讀寫操作，開啟檔案後會回傳一個 File Descriptor (檔案描述器、簡稱 fd)，之後的所有操作都會需要 fd 作為參數。除非每個行程明確將其關閉，否則行程至少會開啟 3 個 fd，分別是 stdin(0), stdout(1) 及 stderr(2)，實際使用這三個 fd 時不需直接用 0 ~ 2 整數值，unistd.h 有預先定義好的 STDIN_FILENO, STDOUT_FILENO 及 STDERR_FILENO。

　　核心內部會替每個行程(task) 維護一份 file table，放在 current->files，用來記錄行程開啟的 fd，可以在 linux/sched.h 的 task_struct中找到他。開檔的系統呼叫會調用 fs/open.c 內的 do_sys_open()，裡面就會註冊 fd 到行程的檔案表，參考以下的核心程式碼：

{% highlight c %}
long do_sys_open(int dfd, const char __user *filename, int flags, umode_t mode)
{
	/* 略... */
	/// 從 current->files 裡面找一個還沒有用到的 fd
	/// current->files 是一個 struct files_struct，定義於 linux/fdtable.h
	/// files_struct 裡面有一個 struct fdtable __rcu *fdt 就是 file table 了
	fd = get_unused_fd_flags(flags);
	if (fd >= 0) {
		/// do_filp_open 進去以後就會調用到 vfs_open 要求檔案系統進行開檔
		struct file *f = do_filp_open(dfd, tmp, &op);
		if (IS_ERR(f)) {
			/// 開檔失敗，把 fd 還回去
			put_unused_fd(fd);
			fd = PTR_ERR(f);
		} else {
			/// notify 機制，通知有檔案打開了
			/// 其他的 notify 也可以在 vfs 層中找到，以 fsnotify_ 作為函數開頭
			fsnotify_open(f);
			/// 將 fd 放到 fdtable 內的 fd array
			fd_install(fd, f);
		}
	}
	/* 略... */
	return fd;
}
{% endhighlight %}

　　詳細結構可參考文末的 files_struct 結構介紹。另外，在預設情況下，子行程會**複製**父行程的檔案表，例如開啟的檔案及存取模式等等，但行程中對檔案的操作不會影響其他行程，例如子行程將檔案關閉後，並不影響父行程的檔案操作。

　

## open() 系統呼叫

　　存取檔案的操作都會需要 fd，而 fd 的取得是透過 open() 系統呼叫。open() 系統呼叫有以下兩種形式，其回傳的 int 變數就是 fd：

{% highlight c %}
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *name, int flags);
int open(const char *name, int flags, mode_t mode);
{% endhighlight %}

　　open() 的 flags 可以是一個或多個值 OR 的結果，用以表示開啟要求的行為，且必須包含 O_RDONLY、O_WRONLY 或 O_RDWR 三者其中之一，但如果開檔的行程不具備對應的操作權限，當然也就不能以該模式開啟檔案。

#### 必要旗標

* O_RDONLY: 以唯讀模式開啟
* O_WRONLY: 以唯寫模式開啟
* O_RDWR: 以讀寫模式開啟

#### 行為操作旗標

* O_APPEND: 以附加模式開啟
	* 以附加模式開啟的檔案，寫入操作都會從檔案末端開始，就算第二個行程也對該檔案進行寫入因而改變了檔案末端位置
	* 例如多個行程要對相同 log 檔進行寫入的應用時，這個模式就會相當好用，因為它可以避免多個寫入者互相競爭的情況
* O_CLOEXEC: 在執行 execl 後自動關閉該 fd
	* fork() 後執行 execl() 是常見的操作，不過 fork() 的子行程會複製父行程的 fd
	* execl() 時通常都不想保留 fd，設立此 flags 的 fd 可以在執行 execl 時自動關閉
* O_NOATIME: 進行*讀取*操作時不更新檔案的存取時間
	* 對於檢索、備份類會經常大量讀取系統上所有檔案的程式有很大的幫助，可以減少讀取後要更新 inode 造成的寫入量
* O_TRUNC: 若開啟的檔案存在且是一般檔案，開啟後就將其截短成長度為 0
	* 以 O_RDONLY | O_TRUNC 開啟檔案是未定義行為

#### 同步、非同步旗標

* O_SYNC: 以 SYNC 模式開啟檔案 (之後會再補充同步 IO)
* O_ASYNC: 如果指定的檔案變成可讀或可寫的狀態，就產生一個 Signal (預設為 SIGIO)
	* 用於 Socket、FIFO、Terminal 等需要非同步操作的情境
	* 不能用於一般檔案
	* 比較常見的應該是使用 aio 相關函式庫而不是使用 O_ASYNC 模式
* O_NONBLOCK: 以 non-blocking IO 模式開啟
	* 對 FIFO 做 read() 時，如果沒有資料可以讀取時立即返回(不阻擋)
	* 因為沒資料而返回時會將 errno 設為 EAGAIN
* O_DIRECT: 以 Direct IO 模式開啟檔案 (之後會再補充 Direct IO)

#### 建檔相關旗標

* O_CREAT: 如果指定的檔案不存在，就建立一個
* O_EXCL: 有指定 O_CREAT 時才有效。如果開檔時檔案已經存在，就回傳失敗，可以避免兩個行程想同時建檔的競爭情況

#### 可能比較少用的旗標

* O_DIRECTORY: 如果開的檔案不是一個目錄就回傳失敗
* O_LARGEFILE: 採用 64 位元 Offset 開啟檔案，可開啟 2GB 以上大檔，在 64 位元系統是預設值
* O_NOCTTY: 和終端機相關
* O_NOFOLLOW: 如果開啟的檔案是 symbolic link，就開啟失敗，但開啟的檔案所屬的目錄是 symbolic link 的話仍然會成功

#### 新檔案的使用權限

　　新檔案的使用權限是由 open 的第三個引數 umode_t mode 做指定。如果沒帶 O_CREAT 旗標，該引數會被忽略，但如果有 O_CREAT 旗標但卻沒有指定 mode 的話，行為會是不明確的。mode 引數就是一般的 Unix 權限位元設定，可以使用 POSIX 預定義的幾個常數用 OR 運算結合指定：

* S_IRUSR: User 擁有 r 權限
* S_IWUSR: User 擁有 w 權限
* S_IXUSR: User 擁有 x 權限
* S_IRWXU: S_IRUSR \| S_IWUSR \| S_IXUSR
* S_IRGRP: Group 擁有 r 權限
* S_IWGRP: Group 擁有 w 權限
* S_IXGRP: Group 擁有 x 權限
* S_IRWXG: S_IRGRP \| S_IWGRP \| S_IXGRP
* S_IROTH: Other 擁有 r 權限
* S_IWOTH: Other 擁有 w 權限
* S_IXOTH: Other 擁有 x 權限
* S_IRWXO: S_IROTH \| S_IWOTH \| S_IXOTH

　　實際上儲存的權限還會受到 umask 影響，是一個行程屬性，他會遮蔽 mode 中對應的權限位元，例如 022 的 mask 會把 0666 的權限變成 0644。相關的處理可以在核心 namei.c 的 lookup_open() 中找到。只要講到**行程屬性**，就有很大的機會會在核心 linux/sched.h 的 task_struct ，而他實際上也是在那裡，位於 task_struct 中的 fs_struct *fs 結構內，umask 系統呼叫也是直接修改該值。

{% highlight c %}
SYSCALL_DEFINE1(umask, int, mask)
{
	mask = xchg(&current->fs->umask, mask & S_IRWXUGO);
	return mask;
}
{% endhighlight %}

　

## creat() 系統呼叫

　　Linux 還有一個系統呼叫是 creat，用來建檔 ([跟 O_CREAT 一樣，他的拼音都少了一個 e](https://en.wikiquote.org/wiki/Ken_Thompson))

{% highlight c %}
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int creat(const char *name, mode_t mode);
{% endhighlight %}

　　他的行為等價於 open()  加上 O_WRONLY \| O_CREAT \| O_TRUNC，他會存在的原因主要是以前的 open() 沒有 mode 引數，我們也可以很容易地在 userspace 內時實作他：

{% highlight c %}
int creat(const char *name, mode_t mode)
{
　　return open(name, O_WRONLY | O_CREAT | O_TRUNC, mode)
}
{% endhighlight %}

　

## close() 系統呼叫
　　應用程式使用完 fd 後，可以用 close() 系統呼叫將 fd 從檔案表中移出，核心主要的實作函式為 fs/file.c 內的 __close_fd()。另外，在 fd 回收完成後，核心也會呼叫檔案的 flush()。

　

## 核心內的 files_struct

　　核心內部的結構大致如下圖，files_struct 內的 fdt 指標會指向目前正在使用的 fdtable 結構，該結構內嵌於 files_struct (fdtab)。每個 fdtable 內都有個 file 陣列，以 fd 為 index 存放每個所需要的 file 結構，預設也是內嵌於 files_struct 的 fd_array，大小為 max_fds：

<figure class="large center">
<img src="/images/2018/12/kernel_files_struct.png" alt="">
</figure>

　　可以看到內嵌在 files_struct 的 fd 陣列大小是固定的 NR_OPEN_DEFAULT，這個值被定義為 __WORDSIZE，根據 32 位元及 64 位元平台的不同，可能是 32 或 64。當開啟的 fd 超過目前的陣列大小時，核心會使用 kvmalloc 重新配置一個更大陣列。

　　另外 fdtable 還有一個 open_fds 陣列，他是以 bitmap 形式記錄哪一個 fd 已經被開啟，當 open() 需要取得未使用的 fd 時，核心便會去掃描 open_fds 陣列內的每一個 bit ，以找到未使用的 fd。

　

## 參考資料

[Robert Love. Linux System Programming](https://www.tenlong.com.tw/products/9789862769812)
