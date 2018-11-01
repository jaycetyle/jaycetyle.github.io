---
layout: post
title: Visual Studio 2013 配置 OpenCV 2.4.9 專案環境
description: "以下筆記如何在 Visual Studio 2013 建置 OpenCV 2.4.9 專案環境，我想這個安裝流程在 2.4.X 的版本應該都是適用，之後的版本應該也是大同小異，沒有很複雜。"
modified: 2014-10-15
tags: [初心者]
categories: [C/C++]
image:
    feature: 
    credit: 
    creditlink: 
---

　　以下筆記如何在 Visual Studio 2013 建置 OpenCV 2.4.9 專案環境，我想這個安裝流程在 2.4.X 的版本應該都是適用，之後的版本應該也是大同小異，沒有很複雜。

<!--more-->

　　首先到 [OpenCV 官方網站](http://opencv.org/) 下載 OpenCV函式庫，根據我翻查以前的資料，每個版本下載下來的東西可能會有不同，不過安裝後有的東西應該差不多，我下載 2.4.9 版本時是一個自解壓縮檔，解壓縮後會有 sources 和 build 兩個資料夾。sources 放的是原始程式碼，我們沒有要修改或編譯他所以可以先不管他，build 內的資料夾就是我們要的東西。

　　doc 資料夾裡面有一個 open_tutorial.pdf ，裡面有很詳盡的教學，包括編譯環境的建置 ( 我寫這篇幹麻？整理翻譯賺文章數嘛 XD)。

　　接著建立環境變數：在我的電腦按右鍵 -> 內容 -> 進階系統設定 -> 進階 -> 環境變數，新增 系統變數
<figure class="center"> <img src="/images/2014/vs2013-opencv-01.png"> </figure>

　　
<div style="line-height:150%">
　　變數名稱: OPENCV_DIR<br>
　　變數值: OpenCV路徑\build\x86\vc12<br>
</div>

　　如果你的是 VS2012，vc12 改成 vc11，如果編譯器設定是 x64 就把 x86 改成 x64，以此類推。接著找到 Path 環境變數，按編輯，在變數值的最後方加上 ; %OPENCV_DIR%\bin (每個變數是使用 ; 進行區隔)
<figure class="center"> <img src="/images/2014/vs2013-opencv-02.png"> </figure>

　　這個步驟主要是要把 OpenCV 的 dll 檔路徑設定成 Windows 搜尋 dll 檔時的預設路徑，當然把 dll 檔和執行檔放在同一個資料夾內也是可行的辦法，但是 OpenCV 的 dll 檔案數量多而且也蠻大的，如果要開發多個軟體，會導致到處都有大量的 dll 檔，比較麻煩。另外，由於 Visual Studio 是在軟體啟動時讀取電腦的環境變數，所以修改環境變數後，Visaul Studio 要重新啟動才有效。

　　接著打開 Visual Studio C++ 專案，在專案名稱按右鍵 -> Property -> Configuration Properties -> C/C++ -> General -> Additional Include Directories 加入 $(OPENCV_DIR)\..\..\include
<figure class="center"> <img src="/images/2014/vs2013-opencv-03.png"> </figure>

　　Linker -> General -> Additional Library Directories 加入 $(OPENCV_DIR)\lib
<figure class="center"> <img src="/images/2014/vs2013-opencv-04.png"> </figure>

　　Linker -> Input -> Additional Dependencies 加入

<div style="line-height:120%">
　　opencv_core249d.lib<br>
　　opencv_imgproc249d.lib<br>
　　opencv_highgui249d.lib<br>
　　opencv_ml249d.lib<br>
　　opencv_video249d.lib<br>
　　opencv_features2d249d.lib<br>
　　opencv_calib3d249d.lib<br>
　　opencv_objdetect249d.lib<br>
　　opencv_contrib249d.lib<br>
　　opencv_legacy249d.lib<br>
　　opencv_flann249d.lib<br>
</div>
<figure class="center"> <img src="/images/2014/vs2013-opencv-05.png"> </figure>

　　可以看到上面的檔案名稱最後的結尾都是 d，表示這些檔案是用於 debug 的編譯設定，如果用於 release 的編譯設定，檔案名稱要換成沒有 d 的版本。249則是版本號，如果使用的版本不同也要更改。

　　$(OPENCV_DIR) 是剛剛設定的系統變數名稱，存的就是我們所指定的 OpenCV 安裝路徑。設定 OPENCV_DIR 系統變數，是為了讓專案的屬性設定與電腦安裝 OpenCV 檔案的路徑盡量不要相依，避免專案設定換台電腦就會失效，在多人開發時會產生很多麻煩。另外由於環境變數只能指定一個路徑，如果你有多個 Visaul Studio 版本就得多設定幾個環境變數，是不怎麼方便，如果有知道其他辦法的麻煩留言告知給我。

　　到這裡就完成了。如果要測試是否有設定成功，可以參考 [葉正聖老師的教學網站](http://www.cmlab.csie.ntu.edu.tw/~jsyeh/wiki/doku.php?id=%E8%91%89%E6%AD%A3%E8%81%96%E8%80%81%E5%B8%AB:%E6%95%99%E7%A0%94%E7%A9%B6%E7%94%9F%E5%AD%B8opencv#step_1最簡5行秀圖程式) 裡面有幾個不到 10 行程式碼的簡單小程式，可以測試看看！

### 後記

　　最近因為看到 <[程式設計師的時光傳話筒：過去的我，希望你能夠在工作前學會這 9 件事！](http://buzzorange.com/techorange/2014/08/13/9-things-i-learned-as-a-software-engineer/)> 這篇文章有談到，如果對程式已經有一定程度的能力 (至少不是剛入門)，可以玩玩計算機視覺、自然語法編寫等等工具或新玩意。我一看到計算機視覺馬上就聯想到 Open CV 這個開源視覺程式庫，身邊也蠻多人有在利用他開發一些視覺系統，不過我到現在都還沒有親自摸過，所以才決定來試試看，之後再找個範例練習一下。