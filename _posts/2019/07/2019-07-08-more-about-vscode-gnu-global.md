---
layout: post
title: VS Code GNU Global 相依專案路徑設定
description: "GNU Global 具備將 Tag file 分散在不同 Project 的功能，並透過相依專案設定，搜尋位於其他專案的 symbol。這在某些大型專案非常有用，因為他可以有效加速 Tag 搜尋和更新的速度。這篇會介紹 gnuGlobal.libraryPath 及 gnuGlobal.objDirPrefix 這兩個好用的設定要怎麼用"
modified: 2019-7-13
tags: [小法師]
categories: [開發環境, C/C++]
image:
    feature: 
    credit: 
    creditlink: 
---

　　在 [VS Code + GNU Global - 打造 Trace Linux Kernel 環境]({% post_url 2018/10/2018-10-29-vscode-gnu-global %}) 這篇我有介紹我改造的 VS Code GNU Global 套件。當初會選擇 GNU Global 作為主要的 tagging engine，除了他的速度很快以外，還有另一個的原因是 GNU Global 具備將 tag files 分散在不同 Project 的功能，再透過相依性設定搜尋路徑。

　　分散 Tag File 在某些大型專案非常有用，他可以有效加速 Tag 搜尋和更新的速度。另外他也支援將 tag files 放在專案目錄以外的地方，以下會分別介紹這兩個功能在 VS Code 中要如何設定。

<!--more-->　

## 打開 VS Code 設定檔

　　第一次接觸 VS Code 的人對於 VS Code 的設定檔一定很陌生，VS Code 的設定都是透過 json 格式來撰寫的。如果不知道設定檔在哪的話，可以從 File > Preferences > Settings > Workspace > Entensions > C/C++ GNU Global > Gnu Global: Library Path，點擊 "Edit in settings.json"，會打開一個 json file。這個 json file 就位於專案目錄下的 .vscode/settings.json，以後可以直接打開他編輯即可。

<figure class="center">
<img src="/images/2019/07/vscode-gnu-global-setting.png" alt="">
</figure>

　

## 設定相依專案 (gnuGlobal.libraryPath)

　　假設你現在有兩個專案，一個位於 /Projects/DocParser，另一個專案位於 /Projects/StrLib 的專案，其中 DocParser 有使用到一些 StrLib 的函式，因此你會希望 Trace DocParser 的時候可以跳轉過去，那該怎麼做呢？

1. 首先替這兩個專案建立 Tag Files。個別打開這兩個專案後，按 F1 叫出命令列後執行 `Global: Rebuild Gtags Database`，來建立 GTAGS 檔案

2. 打開 DocParser 的 settings.json，加入 "gnuGlobal.libraryPath"，內容輸入相依專案的**絕對路徑**，如下:

{% highlight json %}
{
    "gnuGlobal.libraryPath": ["/Projects/StrLib"]
}
{% endhighlight %}

　　這樣子就完成設定了，"gnuGlobal.libraryPath" 接受的是字串陣列，如果你的專案相依於多個專案時，在陣列內用 ',' 隔開多個路徑即可。

　　如果你有非常多的專案，而且你也有辦法取得專案的相依性的話，這個設定是可以寫 Script 來產生的 (我相信我的同事會我在說什麼)，當然 GTAGS 等檔案也是可以寫程式去執行。

　

## 設定 Tag Path Prefix (gnuGlobal.objDirPrefix)

　　這是我最近增加的一個非常好用的功能，也是拜這次改版所賜，套件總算突破2萬下載！

　　好了不廢話，"gnuGlobal.objDirPrefix" 這個設定對於不想把 GTAGS 放在專案目錄的人很有用。他只支援 Unix 檔案系統，因此目前 Windows 版的是不支援的。這個設定接受一個絕對路徑，例如以下:

{% highlight json %}
{
    "gnuGlobal.objDirPrefix": "/Tags/"
}
{% endhighlight %}

　　當然 /Tags/ 這個目錄必須要先手動建立。之後執行 `Global: Rebuild Gtags Database` 建立 GTAGS 等檔案時，他會把你的專案目錄的絕對路徑前面加上這個 Prefix，並且把 GTAGS 產生在裡面。

　　例如你的專案路徑位於 /Projects/DocParser，你的 GTAGS 檔案就會產生在 /Tags/Projects/DocParser/ 中，之後搜尋符號的時候也是會到該目錄搜尋，這樣 GTAGS 就不會汙染你的專案目錄囉！

　　這個功能的實作是使用 GTAGSOBJDIRPREFIX 這個環境變數，想要用 Script 來自動對大量 Project 建立 GTAGS 的朋友們，可以直接設定這個環境變數，然後再下 gtags 指令，詳細說明可以參考 [GNU Global 的說明文件](https://www.gnu.org/software/global/globaldoc_toc.html)。
