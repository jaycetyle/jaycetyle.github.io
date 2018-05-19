---
layout: post
title: 升級囉！Ubuntu 18.04
description: "Stable Ubuntu 18.04 LTS 在 4/26 正式 release 了，根據歷年 Ubuntu 的習慣，這次的版本代號不意外的是 B 開頭的 Bionic Beaver (仿生河狸?)，系統升級的部分據說會在第一個 patch，也就是大約一個月後釋出。以下筆記如何手動搶先從 16.04 升級上去。"
modified: 2018-04-28
tags: [初心者,作業系統]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　Stable Ubuntu 18.04 LTS 在 4/26 正式 release 了，根據歷年 Ubuntu 的習慣，這次的版本代號不意外的是 B 開頭的 Bionic Beaver (仿生河狸?)。我沒有經歷過 14.04 升 16.04 的階段，系統升級的部分據說[會在第一個 patch，也就是大約一個月後釋後釋出](https://askubuntu.com/questions/1028624/not-able-to-upgrade-from-16-04-lts-to-18-04-lts)。

　　不過我今天沒什麼特別的事，就手動把[USB Ubuntu 環境]({% post_url /2018/03/2018-03-24-ubuntu-on-usb %})升級了一下，反正檔案都放在電腦硬碟或[NAS](https://www.synology.com/zh-tw)上，環境爛掉大不了重裝一下就好。以下筆記如何手動搶先從 16.04 升級上去。

<!--more--> 

## 升級流程
　　我是用 Ubuntu 內建的軟體更新來升級，不過在還沒正式開放的情況下，直接打開軟體更新是找不到可升級的新版本的，可以打開 terminal 輸入以下命令:
{% highlight bash %}
$ update-manager -cd
{% endhighlight %}

　　參數 c 是確認是否新發行版可以更新，參數 d 則是更新到 development release，等他搜尋完會出現以下畫面，點下升級吧！
<figure class="large center">
	<img src="/images/2018/04/ubuntu-1804-upgrade-0.png" alt="">
</figure>

　　然後會進到發行版升級的畫面，等他跑一下
<figure class="center">
	<img src="/images/2018/04/ubuntu-1804-upgrade-1.png" alt="">
</figure>

　　確認一下升級內容，看起來會多吃掉 1GB 左右，另外有幾個套件會被移除，列在詳情裡面，稍微看了一下，嗯 ... 我覺得沒差，大不了再裝就有，直接按下開始升級
<figure class="large center">
	<img src="/images/2018/04/ubuntu-1804-upgrade-2.png" alt="">
</figure>

　　過程中彈了不少 python3 相關的升級錯誤我沒有截圖下來，不過 python3 是一個很廣泛的東西，所以我沒有很擔心，只是要一直手動按關閉很煩，按一按等他跑完就可以重開機了。進到新桌面後果然有點不習慣，馬上裝一下 tweak-tool 來調整，Ubuntu 18.04 改成 gnome 了，輸入以下指令安裝
{% highlight bash %}
$ sudo apt install gnome-tweak-tool
{% endhighlight %}

　　稍微調整以後，選擇了 Adwaita-dark 的主題，介面如下，用起來還算順手
<figure class="center">
	<img src="/images/2018/04/ubuntu-1804-upgrade-4.png" alt="">
</figure>

　　後來試了一下 python3，果然沒猜錯，keras 和 tensorflow 等 python package 被刪掉了，不過這些再裝就有了，如果有在用這些套件的可以在升級完以後確認一下
