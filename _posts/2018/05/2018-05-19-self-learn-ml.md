---
layout: post
title: 業餘自學 Machine Learning 經驗分享
description: "這幾年人工智慧 (AI)、機器學習 (Machine Learning) 是非常紅的話題，這些對我來說一直就像是個黑盒子，在學校沒學過，今年年初的時候下定決心要來研究一下，以下分享一些學習經驗給有興趣的業餘人士參考"
modified: 2018-05-19
tags: [小法師,機器學習]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　這幾年人工智慧 (AI)、機器學習 (Machine Learning) 是非常紅的話題，這些對我來說一直就像是個黑盒子，在學校沒學過，新聞和網路上的資訊又很複雜，很容易被誤導，在今年年初的時候下定決心要來研究一下。但從學校畢業以後，要學這些新技術只能靠自己了，以下分享一些學習經驗給有興趣的**業餘人士**參考，揭露 Machine Learning 的神秘面紗。

<!--more-->　

## 學習教材
　　學校的課程還是比較扎實的，我推薦[台大李宏毅的機器學習課程，有線上影片及教材](http://speech.ee.ntu.edu.tw/~tlkagk/courses.html)，照他的課綱看影片就可以了，雖然會花不少時間但務實很多。作業部分我沒寫報告(也沒人可以幫你改)，但**程式一定要自己實作過**，會很有感覺。

　　課程中間老師會開始讓大家使用 Machine Learning 的 Framework，主要是 Keras 搭配 tensorflow backend，我也很推薦使用這套，不到 100 行的 python 就可以訓練出一個很不錯的模型，[Keras 中文文檔](https://keras.io/zh/)有一些基本的說明，[keras 的 github](https://github.com/keras-team/keras/tree/master/examples) 上也有範例模型可以給各位參考，有基礎 python 底子的人要學會應該是很快的。
<figure class="large center">
<img src="/images/2018/05/keras-tensorflow.jpg" alt="">
</figure>
　▲ [圖片來源: Keras blog](https://blog.keras.io/)

　

## 重點提示
　　李宏毅課程裡面有幾堂課是比較偏向 tips 的，例如 Where does the error come from? 和 Tips for deep learning 等，包含什麼是 overfitting、underfitting 以及如何處理的一些技巧，是蠻重要的基礎概念，寫練習題時可以回來消化一下，對實作過程會有所幫助。
<figure class="center">
<img src="/images/2018/05/learning_rate.jpg" alt="">
</figure>
　▲ 學習曲線，圖片來源為李宏毅課程教材

　

## 環境準備
　　工程科目的學習光看書或讀理論通常都是不夠的，一定要自己寫過練習題才會有感覺。環境的話，建議還是有一台安裝好 Linux 作業系統的電腦是比較方便一些，不過也有朋友是在 Windows 上面練習，所以我想應該都可以。

　　另外建議電腦硬體要有 **tensorflow 支援的 GPU**，現在主流的 NVIDIA 顯卡應該都可以 (主要是要有支援 CUDA)，以我目前只有 CPU 訓練的情況來看，訓練一個不算大的神經網路就耗費數小時，如果沒訓練好又要修改模型重新訓練，真的太花時間了，有好的 GPU 可以改善非常得多。而 AMD 顯卡在撰寫這篇文章的時間點來看，還是比較難上手，主因是 tensorflow 對 opencl 的支持程度還沒有很高，至少我花了一段時間還沒有真正用他加速成功，難怪 AMD 都比較喜歡出挖礦卡 ...

　

<figure class="large center">
<img src="/images/2018/05/Jensen_Huang_Computex_Taipei.jpg" alt="">
</figure>
　▲ 老黃你是對的，是我信仰不夠沒有買你家顯卡 Orz [(圖片來源)](https://www.flickr.com/photos/nvidia-taiwan/27275077372)