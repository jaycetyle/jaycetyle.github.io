---
layout: post
title: Git 版本控制筆記 - 使用 github 及 ssh 金鑰設定
description: "git 可以只作為個人版本控制用途，但更多的使用情況是另有一個主要的伺服器保管程式碼，這篇文章會介紹如何使用 github 這個時下最夯的 git 託管服務，將修改好的變更推送 (push) 到遠端以及拖拉 (pull) 本地端，另外也會介紹一下如何設定 ssh 金鑰，讓 github 授權上傳程式碼。"
modified: 2018-03-15
tags: [小法師]
categories: [版本控制]
image:
    feature: 
    credit: 
    creditlink: 
---

　　git 可以只作為個人版本控制用途，但更多的使用情況是另有一個主要的伺服器保管程式碼，這篇文章會介紹如何使用 github 這個時下最夯的 git 託管服務，將修改好的變更推送 (push) 到遠端以及拖拉 (pull) 本地端，另外也會介紹一下如何設定 ssh 金鑰，讓 github 授權上傳程式碼。

<!--more-->　

## 在 github 建立一個新的 repository
　　首先要到 github 申請一個帳號，這應該不困難所以我就沒有建說明了。對於現在軟體工程師而言，github 幾乎快要變成個人名片，所以我很建議可以申請有代表性一點的帳號名稱，github 也是未來找工作時很好展示作品的地方。

　　申請完 git 帳號以後就可以來建立 repository，也就是版本管理的儲存庫。首先將首頁個人頭像旁邊的加號點開，點選 New repository
<figure class="large center">
	<img src="/images/2018/02/github-new-repo.png" alt="">
</figure>

　　在以下頁面輸入 repository 的名稱以後再按 Create repository，就可以建立一個新的 repository 了
<figure class="large center">
	<img src="/images/2018/02/github-new-repo-conti.png" alt="">
</figure>

　　注意到建立版本庫的時候也可以選擇 public 和 private，不過 github 只有學生和付費使用者可以建立私人的儲存庫，不過個人使用沒有要營利的話，公開代碼往往好處大於壞處，所以選擇 public 就可以了。

　

## 設定 ssh 金鑰
　　在我們正式上傳到 github 以前，我們需要先設定 ssh 金鑰。為什麼要設定這個金鑰呢？這是為了讓 github 知道，拿著這個金鑰的操作者是屬於那一個帳號，以賦予上傳等等的權限。如果是第一次使用 ssh key，會需要先產生 ssh 的金鑰，在 Bash 輸入以下指令以產生金鑰
{% highlight bash %}
$ ssh-keygen                                   # 產生金鑰
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jaycelin/.ssh/id_rsa):   # 金鑰存放路徑，直接按 Enter
Created directory '/home/jaycelin/.ssh'.
Enter passphrase (empty for no passphrase):    # 密碼，可設定可不設定，設定的話每次上傳會多需要輸入一次密碼
Enter same passphrase again:                   # 再輸入一次密碼
The key fingerprint is:                        # 之後會顯示你的 fingerprint，到這裡就完成 key 的產生了
{% endhighlight %}

　　金鑰生成以後會放在剛剛前面的金鑰存放路徑，我們要複製 id_rsa.pub 的內容到 github，可以使用 cat 的讀檔指令 :
{% highlight bash %}
$ cat /home/jaycelin/.ssh/id_rsa.pub           # 讀取檔案內容
ssh-rsa ~一連串金鑰~ jaycelin@HOME-PC           # 從 ssh-rsa 複製到 username@pc-name
{% endhighlight %}

　　接著到 github 的設定頁面將剛剛複製的金鑰輸入進去，參考下圖流程
<figure class="center">
	<img src="/images/2018/02/github-settings.png" alt="">
</figure>
---
<figure class="center">
	<img src="/images/2018/02/github-new-sshkey.png" alt="">
</figure>
---
<figure class="center">
	<img src="/images/2018/02/github-enter-sshkey.png" alt="">
</figure>

　　輸入完畢後就完成金鑰設定了。對於不同的 git server，例如 bitbucket 或 gitlab 等等，流程都是類似的，只是匯入金鑰的網頁介面稍有不同而已，重點就是要將 id_rsa.pub 的內容提供給伺服器。

　

## 第一次推送程式碼到 github
　　到這邊我們已經完成　github 的設定了，我們回到剛剛新建的 repository，會發現 github 已經告訴我們該怎麼做了，這邊假設我們已經在本地初始化一個儲存庫，先 cd 到已經準備好要推送的儲存庫資料夾，接著貼上 github 所提示的指令:
<figure class="center">
	<img src="/images/2018/02/github-first-push.png" alt="">
</figure>

　　以下說明這些指令在做什麼，以及一些常用的操作指令 :

**　1. git remote add origin git@URL**
:	git remote add 是新增遠端儲存庫的指令，origin 遠端庫的名稱，git 的慣例是主要追蹤的儲存庫都會叫 origin，當然取別的名字也可以。後方帶的是遠端庫的路徑，除了可以帶網址以外，如果想要把本機上其他的儲存庫當作遠端也是可以的。git 支援同時追蹤多個遠端版本庫，因此也沒有限制只能加一個。

**　2. git push origin master**
:	git push 是推送指令，這條指令的意思是，推送(上傳)本地的 master 分支到 origin 遠端版本庫，同時遠端的分支也是叫做 master。

**　3. git push \-u origin master**
:	-u 代表推送時同時設定 upstream，也就是預設的追蹤遠端，之後要推送的時候就可以只打 git push，省掉 origin 和 master。但我一般還是建議全部都打，比較清楚自己到底推到哪裡去。

**　4. git pull origin master**
:	當遠端版本庫有更新時，可以使用 git pull 將更新拉下來，有點類似 svn 的 update。

**　5. git clone git@URL**
:	git pull 是已經有正在追蹤遠端的儲存庫時，但如果遠端是已經有紀錄的儲存庫，而本地又沒有正在追蹤的儲存庫時，可以使用 git clone 指令把遠端的儲存庫複製下來，之後就一樣是 push pull 的操作了。

　　git 的每條指令都還會有不同參數提供不同功能和變化，但以上是最常使用的幾個，熟練以後若有其他需求可以再去搜尋，大部分 git 都很能夠滿足需求。