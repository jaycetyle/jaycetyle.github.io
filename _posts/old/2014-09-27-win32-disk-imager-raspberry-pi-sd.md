---
layout: post
title: 用 Win32 Disk Imager 備份和燒錄 Raspberry Pi 的 SD 卡
description: "用 Win32 Disk Imager 備份和燒錄 Raspberry Pi 的 SD 卡"
modified: 2014-09-27
tags: [初心者, 工具軟體]
categories: [工具軟體]
image:
    feature: 
    credit: 
    creditlink: 
---

　　不少網站已經介紹過如何使用 Win32 Disk Imager 燒錄 Raspberry Pi 的 SD 卡。但實際上，這套軟體還有讀取 SD 卡的內容並轉成 img 檔的備份功能，玩 Pi 玩一玩怕把穩定的系統玩壞的時候，便可以利用此功能把現在的 SD 卡備份下來，真的出問題的話再重新燒進去就可以還原了，而且此軟體是 Windows 工具，對 Windows 使用者而言也相當方便親民。

<!--more-->

　　沒有此軟體的可以到 [SourceForge](https://sourceforge.net/projects/win32diskimager/) 下載這套軟體並安裝，看到這個網站就知道這套軟體會是免費而且開放原始碼，以下說明備份和燒錄的操作流程

### 備份 SD 卡

1. 點選開啟檔案圖示
    <figure class="large center"> <img src="/images/old/win32-disk-img-01.png" alt=""> </figure>

2. 雖然視窗標題上面寫的是 Select a disk image ，但我們的工作是要備份 SD 卡，當然沒有 image 囉，不過沒關係，找到要的存檔路徑後，直接輸入想要儲存的檔案名稱，之後按開啟舊檔。
    <figure class="large center"> <img src="/images/old/win32-disk-img-02.png" alt=""> </figure>

3. 插入 SD 卡並確認 SD 卡的磁碟代號，在 Device 下拉選單中選擇對應的磁碟代號。都確認無誤後就可以按下 Read 鍵，從 Device 中讀取資料並寫入 img 檔。
    <figure class="large center"> <img src="/images/old/win32-disk-img-03.png" alt=""> </figure>

4. 讀取完畢後會顯示 Read Successfully 視窗到資料夾確認檔案是否已經順利儲存，如果有缺少副檔名情況，可以手動補上 .img 副檔名，這樣子 SD 卡就備份完成囉。

　　

### 燒錄 SD 卡

　　燒錄的流程基本上和備份的流程是相同的，在備份的步驟 2 不要輸入自訂的檔案名稱，而是選擇 img 檔，步驟 3 選擇 Write 。之後會跳出警告 SD 卡檔案可能會被破壞的警告視窗，按確認直接開始燒錄就行了！完成後把 SD 卡插入 Pi 裡面試試看吧！