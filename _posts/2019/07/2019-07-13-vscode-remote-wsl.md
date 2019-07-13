---
layout: post
title: Visual Studio Code Remote - WSL 安裝教學
description: " Visual Studio Code Remote - WSL可以讓 VS Code Server 實際執行在 WSL 裡面，只留 UI 介面在 Windows。這對某些插件非常有用，因為有些東西跑在 Linux 環境是比較容易的。另外 Visual Studio Code Remote 系列還包含 Remote - SSH 模式，這東西就更猛了，如果你的 Build Machine 是遠端的 Linux Server ，他可以直接透過 SSH 跑在 Linux Server 端，像是檔案搜尋等動作，直接執行在遠端 Linux 就會比透過 Samba 或 NFS 快上很多。"
modified: 2019-7-13
tags: [小法師]
categories: [開發環境]
image:
    feature: 
    credit: 
    creditlink: 
---

　　最近在思考如何讓[我的插件](https://marketplace.visualstudio.com/items?itemName=jaycetyle.vscode-gnu-global)能夠跑在 WSL 裡面，後來發現了 [Visual Studio Code Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) 這個好東西，它可以讓 VS Code Server 實際執行在 WSL 裡面，只留 UI 介面在 Windows。

　　這對某些插件非常有用，因為有些東西跑在 Linux 環境是比較容易的。另外 Visual Studio Code Remote 系列還包含 Remote - SSH 模式，這東西就更猛了，如果你的 Build Machine 是遠端的 Linux Server ，他可以直接透過 SSH 跑在 Linux Server 端，像是檔案搜尋等動作，直接執行在遠端 Linux 就會比透過 Samba 或 NFS 快上很多。

<!--more-->　

## 環境安裝

　　想要使用 VS Code Remote - WSL，你必須要安裝以下三個東西

1. Visual Stuio Code
2. WSL (Windows Subsystem Linux)
3. Visual Studio Code Remote - WSL 插件

　　嗯 ... 非常直覺，不需要額外安裝什麼。WSL 的部分在 [Windows Subsystem for Linux 安裝]({% post_url 2018/01/2018-01-07-win-subsys-linux %}) 這篇文章有教學。插件安裝也很簡單，打開 Visual Studio Code Extensions 搜尋 Remote - WSL 就可以找到並安裝

<figure class="center">
<img src="/images/2019/07/vscode-remote-wsl-1.png" alt="">
</figure>

　

## 在 Remote - WSL 中開啟專案

　　按下 F1 ，執行 `Remote-WSL: New Window` 可以開啟一個執行在 WSL 的視窗，或是執行 `Remote-WSL: Reopen Folder in WSL`，直接將目前開啟的目錄改在 WSL 中執行。

<figure class="center">
<img src="/images/2019/07/vscode-remote-wsl-2.png" alt="">
</figure>

　　成功開啟專案後，你會發現左下角多了 WSL 的圖示，另外開啟 Terminal 也會直接進到 WSL，真的非常方便！

<figure class="center">
<img src="/images/2019/07/vscode-remote-wsl-3.png" alt="">
</figure>

　

## 在 Remote - WSL 中安裝插件

　　如果有想安裝其他插件的話，Remote - WSL 環境內是需要另外安裝的，畢竟執行環境已經跑到 WSL 裡面了。

　　到插件中心會發現插件被分成了 Local 和 WSL 部分，要把插件安裝到 Remote 也很簡單，點擊綠色的 Install on WSL，他就會安裝在 WSL

<figure class="center">
<img src="/images/2019/07/vscode-remote-wsl-4.png" alt="">
</figure>

　　Remote - WSL 的參數設定也是獨立的，從 File > Preferences > Settings 打開設定頁面，會發現多了 Remote[WSL] 部分，你可以針對 Remote - WSL 有獨立的設定。

<figure class="center">
<img src="/images/2019/07/vscode-remote-wsl-5.png" alt="">
</figure>

　

## 其他問題

#### C++ GNU Global 插件問題

　　如果你是我的插件使用者，你可能會發現他沒辦法在 Remote - WSL 中執行，經過我的調查，有可能是 Remote - WSL 的執行路徑問題，你會需要明確提供 gtags 和 global 的 binary 路徑 :

{% highlight json %}
{
    "gnuGlobal.gtagsExecutable": "/usr/local/bin/gtags",
    "gnuGlobal.globalExecutable": "/usr/local/bin/global"
}
{% endhighlight %}

　　看來想無痛支援 WSL 並沒有這麼容易，這部分我有空會再來研究是為什麼找不到，如果有大大知道的話也歡迎[發 Pull Request](https://github.com/jaycetyle/vscode-gnu-global/pulls)給我。

#### Remote - SSH

　　Remote - SSH 的安裝也是類似的，但 Windows 使用者還會額外需要安裝 OpenSSH，OpenSSH 的安裝可以參考[微軟的 OpenSH 說明文件](https://docs.microsoft.com/zh-tw/windows-server/administration/openssh/openssh_install_firstuse)
