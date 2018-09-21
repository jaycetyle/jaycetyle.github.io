---
layout: post
title: 避免 Firefox 套用 Ubuntu 的暗色主題
description: "因為 Chrome 記憶體吃太兇的關係，最近我改用 Firefox 瀏覽器了，用起來很順的但一直有一個問題：Firefox 似乎會吃 Ubuntu gtk theme 的顏色。這導致某些網站的文字框或下拉式選單會變成暗色，看起來很奇怪，甚至會造成瀏覽障礙，這篇筆記如何複寫 Firefox 套用到的主題顏色"
modified: 2018-07-11
tags: [初心者]
categories: [Ubuntu 作業系統]
image:
    feature: 
    credit: 
    creditlink: 
---

　　因為 Chrome 記憶體吃太兇的關係，最近我改用 Firefox 瀏覽器了，用起來很順的但一直有一個問題：Firefox 似乎會吃 Ubuntu gtk theme 的顏色。這導致某些網站的文字框或下拉式選單會變成暗色，看起來很奇怪，甚至會造成瀏覽障礙，這篇筆記如何複寫 Firefox 套用到的主題顏色。

<!--more-->　

## 修改方法

我以臺灣證券交易所的網站為例(因為他比較明顯)，可以看到一些文字框都變成了暗色，這應該不是網站設計者原始期望的顏色
<figure class="center">
<img src="/images/2018/07/override-firefox-gtk-theme-01.png" alt="">
</figure>
　

在網址列輸入 about:config 進入進階設定，會出現以下警告訊息，點選 "我發誓，我一定會小心的!" 進入設定頁面
<figure class="center">
<img src="/images/2018/07/override-firefox-gtk-theme-02.png" alt="">
</figure>
　

在設定頁面中按滑鼠右鍵->新增->字串，出現新字串設定，名稱輸入 widget.content.gtk-theme-override
<figure class="center">
<img src="/images/2018/07/override-firefox-gtk-theme-03.png" alt="">
</figure>
　

按下確定後，接著輸入字串的值，內容輸入一個淺色系的 Ubuntu 主題，我是使用 Ambiance
<figure class="center">
<img src="/images/2018/07/override-firefox-gtk-theme-04.png" alt="">
</figure>
　

按確定後重新開啟 Firefox，進到同一個網站比對一下看有沒有修改成功～
<figure class="center">
<img src="/images/2018/07/override-firefox-gtk-theme-05.png" alt="">
</figure>

