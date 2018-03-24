---
layout: post
title: 帶著走的開發環境 - 把 Ubuntu 裝到 USB
description: "這篇簡單筆記如何安裝完整的 Ubuntu 到 USB 內 (不僅僅是 Live USB)。因為當初安裝的時候沒有記得很詳細，所以僅列出安裝過程中的主要重點"
modified: 2018-03-24
tags: [小法師,作業系統]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　軟體開發常常會需要 Linux 作業環境，但電腦又想裝 Windows 時候，一個常見的解決方式是在裝 Virtual Box，不過前一陣子覺得這不完全滿足我的需求，因為我有桌電和筆電兩台電腦，出門用筆電，在家用桌電，要兩台電腦共享 VM 就有一點麻煩了，後來決定直接把 Ubuntu 裝在隨身碟裡面，這樣就可以隨時切換我的開發環境，反正程式碼會丟到 github 或其他雲端，隨身碟掛了也沒損失。

　　這篇簡單筆記當初怎麼安裝 Ubuntu 到 USB 內，因為當初安裝的時候沒有記得很詳細，所以僅列出主要重點。
<!--more-->　
## 安裝流程
1. 準備兩隻 USB，一隻做為安裝用的開機碟，需求 4GB 以上。另一隻是要安裝系統用的，我建議是 **USB3.0** 以上的規格，速度才夠快，容量看需求，建議 **32GB** 以上。

2. 製作 Live USB。有很多方法，Google "Ubuntu live usb" 應該就可以找到很多，例如 UNetbootin 之類的軟體都可以，記得是要放到 4GB 那顆，別搞錯了！

3. 用 Live USB 開機並安裝。小心選擇安裝的位置，不要選錯磁碟了。將第一個分割分割為 128 MB 以上的 efi，第二個分割作為 ext4 主分割，mount root (/)。swap 可以不切割，全靠主記憶體，Install 跳出建議 swap 分割時可以無視 (下圖僅供參考)。
	<figure class="large center">
	<img src="/images/2018/03/ubuntu-install.png" alt="">
	</figure>

4. 安裝完後，拔掉 Live USB 重新開機進到新安裝的 Ubuntu，mount sdx1，也就是剛剛切割的 efi partition，確認裡面是否有 EFI 資料夾。

5. 如果沒有 EFI 資料夾 ...
	* 把 /boot/efi/EFI 的資料夾複製到 sdx1
	* 備份 EFI/Boot/bootx64.efi 後，用 EFI/ubuntu/grubx64.efi 覆蓋
	* 確認 /etc/fstab 內的 /boot/efi mount 的 disk UUID 為 sdx1，如果不是的話，用 blkid 指令查詢 sdx1 的 UUID，並將其改掉

6. 打開終端機關閉 UTC 時鐘，避免 Ubuntu 和 Windows 時間不同而錯亂
	{% highlight Shell %}$ sudo timedatectl set-local-rtc 1 --adjust-system-clock{% endhighlight %}

　　

　　**大功告成！**