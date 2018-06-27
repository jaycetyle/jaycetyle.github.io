---
layout: post
title: 將 Ubuntu 家目錄資料夾的語言改為英文
description: "如果 Ubuntu 安裝成中文版的話，家目錄 (home) 內的資料夾名稱也會安裝成中文的，例如「桌面」、「下載」之類的，但對於經常使用 terminal 的使用者來說，要在中英文之間切換很麻煩，以下教學如何將這些目錄的名稱無痛轉換為英文"
modified: 2018-06-04
tags: [初心者]
categories: [作業系統]
image:
    feature: 
    credit: 
    creditlink: 
---

　　如果 Ubuntu 安裝成中文版的話，家目錄 (home) 內的資料夾名稱也會安裝成中文的，例如「桌面」、「下載」之類的，但對於經常使用 terminal 的使用者來說，要在中英文之間切換很麻煩，以下教學如何將這些目錄的名稱無痛轉換為英文。

<!--more-->

　　首先打開 terminal 輸入以下指令

{% highlight Shell %}
$ export LANG=en_US
$ xdg-user-dirs-gtk-update
{% endhighlight %}

　　輸入完以後會出現以下視窗，告知哪些資料夾會被修改，修改成什麼
<figure class="large center">
<img src="/images/2018/06/home-fold-lang.png" alt="">
</figure>

　　按下「Update Names」以後，資料夾名稱就自動被改好了。另外原資料夾內如果有檔案的話，該資料夾內的檔案不會被搬移，且原資料夾會被保留，如果有檔案的話再手動搬移即可。