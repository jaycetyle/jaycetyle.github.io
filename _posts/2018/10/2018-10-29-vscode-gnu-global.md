---
layout: post
title: VS Code + GNU Global - 打造 Trace Linux Kernel 環境
description: "Linux Kernel"
modified: 2018-10-29
tags: [小法師]
categories: [C/C++, Linux Kernel]
image:
    feature: 
    credit: 
    creditlink: 
---

　Linux Kernel 是一個偉大的開源專案，同時也是相當龐大的專案，要 Trace 這麼大的專案總是要準備一點輔助工具才會有效率。這篇主要介紹使用 VS Code 編輯器，加上我改的 C/C++ GNU Global 套件來 Trace Linux Kernel。

<!--more--> 

## 背景說明

　　一般編輯器或 IDE 都會自帶或是外掛一些 C/C++ 的輔助套件，提供自動搜尋並跳轉到函式定義 (Go to definition) 的功能。

　　以 VS Code 為例，最多人使用的應該就是[微軟提供的 C/C++ 套件](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)，但這個套件遇到 Linux Kernel 就很頭痛，因為專案太龐大，剖析程式碼實在太久，自帶的一些好用功能，如修改時自動更新等，會大量吃 CPU 資源而成為累贅。

　　為了解決這個問題，我找了 [C++ Intellisense](https://marketplace.visualstudio.com/items?itemName=austin.code-gnu-global) 這個套件。他使用 [GNU Global](https://www.gnu.org/software/global/) 作為背後的運作核心，和微軟的 C/C++ 插件有兩個主要的不同:

1. C/C++ 套件應該是有使用 clang 之類的 front end 做 parser，而 GNU Global 只是一個簡單的 tagging system，不會去建立抽象語法樹，功能及精確度比較差，不過速度也會快很多。

2. C/C++ 套件背景會執行一個 language server 的服務，當專案太龐大的時候就會變得非常吃資源。C++ Intellisense 套件沒有背景服務，要建立或是更新 tag 只有手動或是在檔案存檔時觸發。

　　簡單來說就是速度快不吃資源，而且也足夠我使用。那為什麼要自己再改一個呢？因為這個套件的作者已經一年多沒有維護了，隨著 VS Code 一直在改版，像是同時開啟多個專案資料夾的功能，該套件也不支援，有些設定後來也壞了，於是下定決心自己改一個發布出來，以下介紹如何安裝該套件。

## 安裝 GNU Global

　　在安裝/使用這個套件前，需要先安裝 GNU Global 6.5 以上的版本 :

#### Linux 使用者
　　Ubuntu 18.04 及之後的使用者可以直接透過 apt 來安裝 :
{% highlight sh %}
sudo apt install global
{% endhighlight %}

　　其他版本的 Linux 使用者可以到 [GNU Global 下載頁面](https://www.gnu.org/software/global/download.html) 去找對應的 package，或是直接下載 source code 來建置。

#### Windows 使用者
　　Windows 的使用者可以在 [Jason Hood 的網頁](http://adoxa.altervista.org/global/) 下載已經建置好的 Windows 可執行檔，將其解壓縮後，將路徑加到環境變數。環境變數要在系統內容做設定，Windows 10 使用者可以直接在工作列搜尋"環境變數"，找到環境變數設定的介面後進行設定：

<figure class="center">
<img src="/images/2018/10/vscode-gnu-global-03.png" alt="">
</figure>　

## 安裝 VS Code GNU Global

　　基本環境準備好後，就可以開始安裝套件了，打開 VS Code 並到的擴展套件頁面搜尋 "GNU Global"，找到 C/C++ GNU Global 這個套件，按安裝並重新載入 VS Code 以啟用套件。
<figure class="center">
<img src="/images/2018/10/vscode-gnu-global-01.png" alt="">
</figure>

　　安裝好以後先按 F1，叫出命令列後執行 `Global: Show GNU Global Version` 確認 GNU Global 有正確安裝及版本號碼。

<figure class="center">
<img src="/images/2018/10/vscode-gnu-global-04.gif" alt="">
</figure>

　　接著同樣叫出命令列後執行 `Global: Rebuild Gtags Database`，這個命令會在專案資料夾內建立三個檔案: GTAGS、GRTAGS、GPATH，這三檔案就是 GNU Global 的 tag 檔 (你會想把他們加入全域的 gitignore 的)，建立完成後就可以對著函數按 **F12** 跳轉到函數定義了。

<figure class="center">
<img src="/images/2018/10/vscode-gnu-global-05.gif" alt="">
</figure>

　　此套件還有另一個功能，當每次檔案進行儲存時，套件會自動更新 GTAGS，因此不需要在每次修改 code 後都手動重建 tag 檔。但這個功能有個限制，如果 GTAGS 檔案過大的話更新的時間會較長，為了避免變成每次存檔都會頓一下，這個功能在 GTAGS 檔案大於 50MB 時會自動被關閉，此時就需要定期重建 tag。autoUpdate 相關設定及其他功能可以在套件的說明頁面中找到。
