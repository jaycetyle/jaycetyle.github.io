---
layout: post
title: Git 版本控制筆記 - 環境設定
description: "Git 版本控制教學筆記 - 環境設定"
modified: 2018-02-04
tags: [小法師,版本控制]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　最近跟一些朋友聊了一下公司軟體開發的情況，發現一些傳統的公司還停留在沒有版本控制，或方法很陽春的情況，因此決定來寫一下 Git 教學筆記，一個我認為即將一統版控江湖的軟體，順便藉此機會讓自己 Git 的使用能夠更深入。這篇文章主要說明 Git 的設定，有一定程度的使用者可以快速瀏覽過去就好。

<!--more-->　

## 建議的學習方式
　　學 Git 最快的方法仍然是**指令操作**輔以 GUI 工具，很多 Windows 使用者會很不適應或排斥指令操作。我也是從 Windows 開始學程式的，也同樣經歷過排斥輸入指令的階段，不過要學好 Git 並感受到 Git 真正**好用**的地方，最終還是要學會指令操作。大部分的教學文應該也會建議從指令開始學，我的建議是就當作是被騙一次試試看，用熟以後是不會後悔的。

　　以下的步驟建議準備一台電腦跟著操作一次，我相信會比單純看文章快非常多。

　

## 環境準備
1\.　安裝 Windows 版 git
:	在 Windows 安裝軟體我想對各位來說應該都不難，先到[官方網站](https://git-scm.com/download/win)下載合適的版本，一直點下一步安裝即可。安裝完以後執行 Git Bash，可以嘗試在裡面輸入圖中的指令，會顯示 Git 安裝的版本。
<figure class="center">
	<img src="/images/2018/02/git-bash.png" alt="">
</figure>

　　之後是我個人推薦的環境: 使用 Ubuntu 子系統。*你可以自行決定是否要採納*，如果是初心者我建議可以跟著試試。使用 Ubuntu 子系統有相當多好處，工具齊全、問題容易 Google，光是以上這兩點應該就值得你試試看了，學會以後連 Linux 都可以順便上手，算是一舉數得。

2\.　[安裝 Windows Subsystem for Linux]({% post_url 2018/01/2018-01-07-win-subsys-linux %})
:	依照[連結的文章]({% post_url 2018/01/2018-01-07-win-subsys-linux %})安裝 Ubuntu 子系統。另外我還推薦安裝 VS Code，一套相當好用的編輯器，如果有使用的話建議依照文末教學將 Terminal 設定成 Ubuntu Bash。

　　
3\.　確認 Ubuntu 子系統的 Git
:	你可以把 Linux 想成他是一個 Virtual Machine，因此他的環境跟外面的 Windows 不一樣。可以用先前教的版本查詢測試看看 git 是否已經安裝在 Ubuntu 內了，如果沒有的話，輸入以下指令安裝 git
{% highlight Shell %}
$ sudo apt-get install git # 如果 Ubuntu 內沒有 git 的話，輸入此指令安裝 git
{% endhighlight %}


4\. 安裝 vim
:	vim 很複雜也很強大，不過我們只需要他來編輯提交訊息，並不困難。在 Ubuntu 子系統內輸入以下命令來安裝 vim
{% highlight Shell %}
$ sudo apt-get install vim
{% endhighlight %}　　
　

## 設定 Git
　　在正式使用 Git 之前，我們需要設定使用者資料，這個資料會在提交程式碼的時候付在提交紀錄(Commit message) 裡面，沒設定的話在第一次提交時 git 也會提醒你要進行設定。進到 Ubuntu Bash (or Git Bash)，輸入以下指令設定使用者資料
{% highlight Shell %}
$ git config --global user.name "YOUR NAME"
$ git config --global user.email "mailto@your.mail"
{% endhighlight %}

　　接著我們還需要設定編輯提交紀錄的編輯器，這裡是指定為 vim
{% highlight Shell %}
$ git config --global core.editor vim
{% endhighlight %}

　　以上的動作在安裝完 git 以後只要執行一次就好。到這裡我們算是把事前工作都準備完成了，已經可以來正式使用囉！

　

## Git 指令縮寫 (別名)
　　git 也有縮寫指令，例如我們可以把 checkout 縮寫成 co，以後想輸入 git checkout 時就只要輸入 git co，可以少打幾個字。以下幾個算是蠻常見的，很多人都會使用相同的設定，也會是之後最常用的幾個指令，可以先設起來
{% highlight Shell %}
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.st status
{% endhighlight %}

　

## 設定檔的儲存位置及 VIM 編輯
　　git 設定檔的位置會儲存在使用者家目錄的 .gitconfig，也就是 ~/.gitconfig，在 Linux 中 ~ 代表使用者的家目錄，通常會是在 /home/your_name 這個位置。我們可以用 vim 嘗試打開他看看
{% highlight Shell %}
$ vim ~/.gitconfig
{% endhighlight %}

　　第一次接觸 vim 通常會很慌亂 「要怎麼退出他？」。其實不用緊張，我們先認識 vim 的兩個模式，第一個是 normal mode，在這個模式下可以輸入一些 vim 的操作指令。第二個模式是 insert mode，在這個模式下打字的話可以編輯內文。我們可以在畫面的最下方看是否有 "Insert" 來判斷是在哪個模式。如果想要從 normal 進到 insert，可以按鍵盤的 a 或是 i，反之，如果要從 insert 模式回到 normal mode，可以按 ESC，如果沒退回去就多按幾下。
<figure class="large center">
	<img src="/images/2018/02/vim-mode.png" alt="">
</figure>

　　那要如何退出 vim 呢？這可是個熱門問題呢！要離開 vim 我們需要切換到 **normal mode** 中輸入指令並按 enter，其中最常使用的指令就是以下三種
{% highlight Shell %}
:q		# 離開
:x		# 存檔並離開
:q!		# 不存檔並離開
{% endhighlight %}

　　輸入的指令會顯示在最下方，可以參考下圖 (email 需要輸入，只是被我馬賽克掉了)
<figure class="half center">
	<img src="/images/2018/02/vim-quit.png" alt="">
</figure>

　　VIM 的功能有很多，不過我們只需要會使用到這個程度就很夠用了。git 的設定也可以直接在 ~/.gitconfig 修改，可以嘗試用 VIM 編輯看看。