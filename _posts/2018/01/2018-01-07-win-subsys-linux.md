---
layout: post
title: Windows Subsystem for Linux 安裝 (Ubuntu bash)
description: "Windows 內建的命令列工具有 cmd 和 Power Shell，如果喜歡 Linux bash，也可以選擇 Cygwin 或 MinGW 等等。現在 Win 10 還有一個更方便的選擇 : Windows Subsystem for Linux (WSL)，可以讓 Ubuntu 的一些實用工具直接在此系統上原生執行。"
modified: 2018-03-27
tags: [初心者]
categories: [開發環境, Ubuntu 作業系統]
image:
    feature: 
    credit: 
    creditlink: 
---

　　Windows 的視窗介面很好用，不過對於開發者而言，有些工作使用命令列工具 (Command Line) 還是方便些。Windows 內建的命令列工具有命令提示字元 (cmd) 和 Power Shell，如果喜歡 Linux bash，也可以選擇 Cygwin 或 MinGW，如果有安裝過 git 的話還有 git bash。以上這些工具我都有試過，現在 **Windows 10** 還有一個更方便的選擇 : Windows Subsystem for Linux (WSL)，可以讓 Ubuntu 的一些實用工具直接在此系統上原生執行。

<!--more-->

　　安裝 WSL 的方法非常簡單，首先使用**系統管理員權限**開啟 **Power Shell** :
<figure class="half center">
	<img src="/images/2018/01/open-powershell.png" alt="">
</figure>

　　執行以下命令 :
{% highlight shell %}
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
{% endhighlight %}

　　執行完以後會要求重新開機，輸入 Y 重開 :
<figure class="center">
	<img src="/images/2018/01/enable-wsl.png" alt="">
</figure>

　　接下來到*灌完 Win 10 以後就幾乎沒開過*的 **Windows Store** ，安裝 Ubuntu :
<figure class="large center">
	<img src="/images/2018/01/winstore-ubuntu.png" alt="">
</figure>

　　我們已經快完成了，執行剛剛安裝好的 Ubuntu ，依照指示輸入使用者名稱和密碼就大功告成！
<figure class="large center">
	<img src="/images/2018/01/init-ubuntu.png" alt="">
</figure>

　　下面是提供給使用 VS Code 的開發者， VS Code 有一個非常好用地方是整合了終端機，如果想把終端機換成 Ubuntu 子系統的話，可以在 File > Preferences > Settings 內加上
{% highlight json %}
"terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\bash.exe"
{% endhighlight %}

　　按快捷鍵 Ctrl + `(tab 上面那個按鈕) 把終端機叫出來，此時終端機已經換成剛剛安裝好的 Ubuntu Subsystem 囉！



