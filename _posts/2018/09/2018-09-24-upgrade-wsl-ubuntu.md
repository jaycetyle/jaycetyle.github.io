---
layout: post
title: 更新 WSL 內的 Ubuntu 到 18.04
description: "Stable Ubuntu 18.04 LTS 正式 release 以後，大家應該陸陸續續開始從 16.04 升上去了，不過 WSL (Windows Subsystem for Linux) 內的並不會自動升級，應該也有人發現 Microsoft Store 內多出了 Ubuntu 16.04 LTS 和 Ubuntu 18.04 LTS 兩個 Image，那到底該如何更新呢？"
modified: 2018-09-24
tags: [初心者]
categories: [Ubuntu 作業系統]
image:
    feature: 
    credit: 
    creditlink: 
---

　　Stable Ubuntu 18.04 LTS 正式 release 以後，大家應該陸陸續續開始從 16.04 升上去了，不過 WSL (Windows Subsystem for Linux) 內的 Ubuntu 並不會自動升級，應該也有人發現 Microsoft Store 內多出了 Ubuntu 16.04 LTS 和 Ubuntu 18.04 LTS 兩個 Image，那到底該如何更新呢？

<!--more--> 

## 升級流程
　　[Microsoft Developer 部落格](https://blogs.msdn.microsoft.com/commandline/2018/07/09/upgrading-ubuntu/)有簡單說明了升級的方式。首先，*Ubuntu 16.04 LTS* 和 *Ubuntu 18.04 LTS* 這兩個 Image 是提供給想要安裝指定版本的使用者，Ubuntu 16.04 LTS 會持續維護到 2021 年壽命終止(end of live)，而 Ubuntu 18.04 LTS 則會繼續維護到 2023 年 EOL。

<figure class="center">
<img src="/images/2018/09/wsl-ubuntu-upgrade-01.png" alt="">
</figure>

　　在這之前的使用者應該裝的都是*無印版 Ubuntu* 這個 Image，這個 Image 的版本會持續追蹤之後所有的 LTS 版，不想升級的使用者可以繼續使用，想升級的使用者，[Microsoft Developer 部落格](https://blogs.msdn.microsoft.com/commandline/2018/07/09/upgrading-ubuntu/) 建議的方法是使用 sudo do-release-upgrade 來升級。

　　我的升級流程如下，提供給各位參考：

1. 備份所有重要資料: 裝 WSL 的人應該資料都還是存在 Windows 居多，因此主要是備份一些設定。
2. 輸入以下指令升級:

{% highlight sh %}
$ sudo apt update -y
$ sudo apt upgrade -y
$ sudo apt update -y
$ sudo do-release-upgrade
{% endhighlight %}

　　前三個指令只是順便升級一下已經安裝的套件，避免 16.04 升 18.04 時跳太大。另外在升級過程中會問一些問題看要不要繼續，反正重要資料都備份了，就直接一路 y 下去。
<figure class="large center">
<img src="/images/2018/09/wsl-ubuntu-upgrade-02.png" alt="">
</figure>

　　整個過程大概半小時到一小時，中間大概還手動 y 了 3 ~ 4 次吧？升級完成以後重新開啟終端機，並透過以下指令確認有無升級成功！
{% highlight sh %}
$ jaycelin@JAYCE-NB:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.1 LTS
Release:        18.04
Codename:       bionic
{% endhighlight %}