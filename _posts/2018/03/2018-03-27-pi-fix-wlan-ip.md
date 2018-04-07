---
layout: post
title: 為 Raspberry pi 的 wlan 設定固定的區域網路 IP
description: "我的 Raspberry pi 是透過 USB 無線網卡上網(wlan0)，但家裡 Hinet 的小烏龜如果重開，或是重新開機，IP 有可能會變，連接時就要重新設定 IP，很麻煩！所以我小小研究了一下讓 pi 的 wlan0 IP 可以固定的方法，蠻簡單的，以下說明設定流程"
modified: 2018-03-27
tags: [初心者]
categories: [軟體開發, 樹莓派]
image:
    feature: 
    credit: 
    creditlink: 
---

　　我的 Raspberry pi 是透過 USB 無線網卡上網(wlan0)，但家裡 Hinet 的小烏龜如果重開，或是重新開機，IP 有可能會變，這樣子連接時就要重新設定 IP ，很麻煩！所以我小小研究了一下讓 Raspberry pi 的 wlan0 IP 可以固定的方法，蠻簡單的，以下說明設定流程。

<!--more-->
　　首先，為了避免改壞設定檔，先輸入 sudo cp /etc/network/interfaces /etc/network/interfaces.backup 備份一下，接著輸入 route -n 及 ifconfig 命令來查詢網路的狀態，可以得到以下的畫面
<figure class="large center">
	<img src="/images/2018/03/pi-ifconfig.png" alt="">
</figure>

　　有三個重要的位址已經標記在圖上了，把這 1 和 2 的位址記下來。圖中標記的這三個位址的前三個數字都是 192.168.1.*，你想要指定的 IP 位址前三個數字也要相同。

　　接著輸入 sudo vim /etc/network/interfaces 打開設定檔 (或其他習慣的文字編輯器也可以)，在最後直接加入以下內容，其他的部分不需更改：

    auto wlan0
    iface wlan0 inet static
    address 192.168.1.110   # 你想要指定的 IP 位址
    netmask 255.255.255.0
    gateway 192.168.1.1     # 輸入上圖中 2 的位址
    network 192.168.1.0     # 輸入上圖中 1 的位址
    wpa-essid ****          # 無線網路的 id
    wpa-psk ****            # 無線網路的密碼

　　都設定好後，sudo reboot 重新開機，看看 IP 有沒有變成指定的位址，有的話就是成功了！