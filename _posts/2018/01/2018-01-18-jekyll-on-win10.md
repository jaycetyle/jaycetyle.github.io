---
layout: post
title: 在 Win10 安裝 Jekyll 部落格網站產生器及簡單起步教學
description: "在 Windows 10 安裝 Jekyll 部落格網站產生器，及簡單起步教學"
modified: 2018-01-18
tags: [小法師,網站架設]
categories: [網站架設]
image:
    feature: 
    credit: 
    creditlink: 
---

　　Jekyll 是一個非常方便的靜態部落格框架，可以讓想架部落格的人可以不受平台的限制架設網站，如果有程式基礎的話，他的靈活性也相對比部落格平台高上許多，可以自行修改成具有個人特色的網站。

　　什麼是**靜態網頁**呢？簡單來說就是不包含資料庫等能和使用者互動的單純網頁，這對於單一網頁不常變動的部落格來說是一個不錯的選擇，唯一的缺點是一般部落格都會需要的留言系統會需要資料庫，但這部分已經有相當多平台提供服務，像本站是使用 [Disqus](https://disqus.com/)，其他像是 [Facebook 留言外掛程式](https://developers.facebook.com/docs/plugins/comments?locale=zh_TW) 也是不錯的選擇。

### 要怎麼安裝呢？
　　Jekyll 主要是使用 Ruby 開發的，Ruby 本身有 Windows 版本，但相對麻煩，我個人最推薦的方式還是使用 **Windows Subsystem for Linux** 來安裝 Ruby，WSL 安裝方式可以參考[這篇](https://blog.jaycetyle.com/2018/01/win-subsys-linux/)，這裡安裝的是 Ubuntu 版本，所以在接下來的範例你會看到 *apt-get*。如果你想要使用其他的方式安裝，可以參考 [Jekyll 官方教學](https://jekyllrb.com/docs/windows/)，這篇文章主要也是翻譯自官方教學。如果你懶得看說明，就把從這個段落的的所有指令一行一行複製輸入就好囉。

　　打開剛安裝好的 Ubuntu Terminal，先輸入以下指令更新套件列表
{% highlight shell %}
jaycelin@UBUNTU:~$ sudo apt-get update -y && sudo apt-get upgrade -y
{% endhighlight %}

　　接著安裝要 Ruby，我們先加入 BrightBox 的 ppa，然後就可以直接安裝
{% highlight shell %}
jaycelin@UBUNTU:~$ sudo apt-add-repository ppa:brightbox/ruby-ng
jaycelin@UBUNTU:~$ sudo apt-get update
jaycelin@UBUNTU:~$ sudo apt-get install ruby2.3 ruby2.3-dev build-essential
{% endhighlight %}

　　更新一下 Ruby gems
{% highlight shell %}
jaycelin@UBUNTU:~$ sudo gem update
{% endhighlight %}

　　安裝 Jekyll 和 bundler
{% highlight shell %}
jaycelin@UBUNTU:~$ sudo gem install jekyll bundler
{% endhighlight %}

　　呼叫以下指令看看能不能看到 Jekyll 版本，確認我們有安裝成功
{% highlight shell %}
jaycelin@UBUNTU:~$ jekyll -v
{% endhighlight %}

　　大功告成！(好快?)

### 要怎麼開始呢？
　　從這裡需要會使用 **Git**，如果你還不會用的話，建議先去學一下最基礎的四個指令: git clone, git add, git commit 以及 git push。

　　Jekyll 起步我認為最簡單的方式還是找個樣板來改，他的樣板很多，Google *Jekyll Theme* 就可以找到一大堆，選好 theme 以後通常你也會找到他的 github，最簡單的方式就是把他 clone 下來。

　　我這裡以 jekyll bootstrap 這個常見的樣板做為範例，其他的樣板大多可以依樣畫葫蘆，如果不行的話多注意他的 README 說明，首先先 clone 他
{% highlight shell %}
jaycelin@UBUNTU:~$ git clone https://github.com/llorephie/jekyll-bootstrap.git
jaycelin@UBUNTU:~$ cd jekyll-bootstrap
{% endhighlight %}

　　安裝個 bundle，過程中有可能需要輸入你的 Ubuntu 密碼
{% highlight shell %}
jaycelin@UBUNTU:~$ bundle install
{% endhighlight %}

　　在 Local 端執行以下指令運行看看
{% highlight shell %}
jaycelin@UBUNTU:~$ bundle exec jekyll serve
{% endhighlight %}

　　接著就可以到瀏覽器輸入 localhost:4000 看看運行結果囉！有沒有非常簡單？

### 再更進階一點，我想要把部落格放到 Github 上？
　　Github 有提供一個 Github Pages 的服務，可以把靜態網站放在 Github 上運行，對於本來就有在使用 Github 來說我想這一定不是難事。

　　首先到你的 Github 上建立一個新的 repository，名稱為 **USERNAME.github.com**。注意不要真的用 *USERNAME* 建立內，這裡指的是*你的 github 帳號*。

　　接著把 clone 下來的樣板 remote 換成剛剛建的 repository
{% highlight shell %}
jaycelin@UBUNTU:~$ git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
{% endhighlight %}

　　PUSH 上去
{% highlight shell %}
jaycelin@UBUNTU:~$ git push origin master
{% endhighlight %}

　　大功告成！如果網頁建立有問題的話，你應該會收到 github 寄來的信告知建立狀況，如果沒問題，就可以到 USERNAME.github.io 看看你的網頁運行的成果！

### 參考文獻
Jekyll on Windows 官方教學: https://jekyllrb.com/docs/windows/