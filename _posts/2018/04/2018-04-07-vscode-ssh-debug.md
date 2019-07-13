---
layout: post
title: VS Code + ssh 樹莓派遠端除錯
description: "這篇主要筆記如何運用 VS Code 的 Native Debug 插件，直接遠端到樹莓派(Raspberry Pi)上面的 gdb 進行 C 語言的程式除錯，可以監控變數，設定中斷點，也可以按 F10 單步執行，對於其他 Linux 系統的 embedded system 只要能夠安裝 gdb、ssh 和掛載遠端目錄應該也都適用"
modified: 2018-04-07
tags: [巫師]
categories: [開發環境, C/C++, 樹莓派]
image:
    feature: 
    credit: 
    creditlink: 
---

　　在一個新的環境寫程式之前，我自己的習慣是一定要先摸一下除錯器 (debugger) 的使用，有好的除錯器輔助可以讓軟體開發省下非常多的時間。這篇主要筆記如何運用 VS Code 的 Native Debug 插件，直接遠端到樹莓派上面的 gdb 進行 C 語言的程式除錯，可以監控變數，設定中斷點，也可以按 F10 單步執行，對於其他 Linux 系統的 embedded system 只要能夠安裝 gdb 、 ssh 和掛載遠端目錄應該也都適用。

<!--more-->

## 環境準備

　　首先需要**一台安裝好 VS Code 的開發機**，我選擇的是 Ubuntu，並**設定好 NFS Server** 讓樹莓派可以掛載，而 **export 的目錄就是想要除錯的 source code 位置**。

　　**樹莓派上需要安裝 gdb**，如果沒有安裝的話可以透過 apt install 安裝他，**並掛載開發機上的 source code 目錄**。

　　設定 NFS 的主要目的是，VS Code 的除錯方式是透過 ssh 連進樹莓派，並叫起樹莓派上的 gdb 來除錯，gdb 在除錯的時候會需要程式碼，我的方式是透過 NFS 讓他可以去得開發機上的程式碼。想把程式碼放在樹莓派上也行，NFS Server 和 Client 相反而已，總之只要讓樹莓派和開發機能夠看到同一份程式碼就可以了。Windows 我沒有試過，但只要能夠 ssh 應該就可以，樹莓派則可能要設定 Samba 之類的來共享程式碼。

　　以上環境都準備好以後，**在 VS Code 上安裝 Native Debug 插件**。
<figure class="center">
<img src="/images/2018/04/native-debug-extension.png" alt="">
</figure>

　

## 操作流程

　使用 VS Code 開啟專案目錄，如下圖依序點選 VS Code 的 debugger 設定，並選擇 GDB
<figure class="center">
<img src="/images/2018/04/native-debug-1.png" alt="">
</figure>

　launch.json 內有一些預設的 configuration，選擇GDB: Launch over SSH。如果你沒有這個選項的話，請先確認 Native Debug 插件是否有正確安裝
<figure class="center">
<img src="/images/2018/04/native-debug-2.png" alt="">
</figure>

　Json configurations 主要有以下設定需要修改，我用註解標示
{% highlight json %}
"configurations": [
    {
        "type": "gdb",
        "request": "launch",
        "name": "Launch Program (SSH)",
        "target": "${workspaceRoot}/hello", // 要除錯的執行檔路徑
        "cwd": "${workspaceRoot}",
        "ssh": {
            "host": "192.168.0.101",        // 樹莓派的 IP
            "cwd": "/mnt/helloworld/",      // 樹莓派上的 source code 位置
            "user": "pi",                   // 樹莓派的登入帳號
            "password": "raspberry"         // 樹莓派的登入密碼
        }
    }
]
{% endhighlight %}

　當然，要把想偵錯的執行檔編譯出來，可以使用 cross compiler 或直接用 pi 上帶的 gcc，記得加上 -g 編譯 debug symbol。這樣就搞定了，按 F5 開始進行 debug，可以在行號旁邊點一下設定中斷點，F10 則是單步執行
<figure class="center">
<img src="/images/2018/04/native-debug-3.png" alt="">
</figure>