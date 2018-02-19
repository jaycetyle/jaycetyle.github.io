---
layout: post
title: Git 版本控制筆記 - 在 Git 提交(commit)檔案
description: "這篇文章會說明如何使用 git 初始化一個 Repository (Git 的儲存庫)，並透過 git add、git commit 提交檔案到版本庫裡面"
modified: 2018-02-19
tags: [小法師,版本控制]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　在[上一篇文章](({% post_url 2018/02/2018-02-04-git-startup %}))中我們已經架設好 git 的基本環境，現在可以來開始使用 Git 囉，這篇文章會說明如何初始化一個 Repository (Git 的儲存庫)，並提交檔案到裡面。

<!--more-->　

## 初始化 Repository (git init)
　　要初始化一個具有 git 託管的專案非常簡單，和 SVN 等集中式的版本管理系統不同，git 的版本控管是直接建立在專案目錄中，並不需要實際存在一個伺服器。只要在專案目錄下輸入 **git init** 指令，這個目錄就可以有 git 版本控管了，就算這個目錄不是空的也沒問題。以下我們還是從一個空的專案目錄開始
{% highlight Shell %}
$ cd ~              # cd 是切換當前工作目錄的指令，這條指令是切換到家目錄 (~)
$ mkdir gittest     # mkdir: 建立一個名為 gittest 的資料夾
$ cd gittest        # 切換到 gittest
$ git init          # 初始化 git，讓這個資料夾可由 git 託管
# Initialized empty Git repository in /home/xxx/gittest/.git/

$ ll                # 列舉資料夾
{% endhighlight %}

　　最後一個指令 ll 是列舉資料夾內的檔案，列舉的結果應該會有以下 3 個東西
{% highlight Shell %}
.    : 是指當前目錄，cd . 的話，工作目錄不會改變 (切換到自己)
..   : 是指上層目錄，cd .. 的話，會切換到上層目錄
.git : 這個就是 git 版本控管的資料庫資料夾
{% endhighlight %}

　　建立資料夾的動作要在 Windows 中操作也沒問題，只要 cd 到想要做練習的資料夾中輸入 git init 便可初始化 git。如果想要解除 git 託管，只要把 **.git** 這個資料夾刪除，*但同時也會失去所以版本控管的歷史紀錄*，要小心操作。

　

## 提交檔案到 git 儲存庫
　　我們先建立一個檔案作為之後提交的練習，這個檔案可以直接用 Windows 建立，或者是輸入以下指令建立一個 README.txt 檔案
{% highlight Shell %}
$ echo "git practice" > README.txt
{% endhighlight %}

　　在真正操作 git 之前，我想先介紹 git 的暫存區域。這個是 git 和許多版本控管系統不同的地方，git 在提交之前，必須要先把檔案的「變更」加入(add)到暫存區域(staging area)，這個變更才能被提交(commit)到版本儲存庫。

　　注意一個關鍵，雖然在之後操作指令時是輸入檔案名稱，但實際加入的是檔案的「變更」，因此**檔案加入到暫存區域後有變更的話還要再加入一次**，否則之後的變更不會被提交，參考下圖
<figure class="large center">
	<img src="/images/2018/02/git-staging-area.png" alt="">
</figure>

　　現在我們知道變更要先加入到暫存區域。在加入之前，我們先用 git status 看一下目前的 git 狀態
{% highlight Shell %}
$ git status
# On branch master
# No commits yet
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         README.txt
{% endhighlight %}

　　可以看到 Untracked files 裡面有 README.txt，這代表目前這個檔案還沒有被 git 追蹤，他也提示了使用 git add 加入想要提交的檔案到暫存區域，我們現在就加入看看
{% highlight Shell %}
$ git add README.txt
$ git status
# On branch master
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   README.txt
{% endhighlight %}

　　現在 README.txt 已經變成 **Changes to be committed**，而且是 **new file**，使用 git commit 將變更提交到儲存庫
{% highlight Shell %}
$ git commit
{% endhighlight %}

　　如果有照上一篇文章的設定做的話，此時會進入到 vim，按 a 或 i 進入 insert 模式編輯提交的紀錄，其中單行 # 開頭的是註解，不會進入提交訊息之中。輸入完成後按 ESC 退出 insert 模式並輸入 :x 存檔離開，這樣便完成了一次提交
<figure class="large center">
	<img src="/images/2018/02/git-commit-message.png" alt="">
</figure>

　　使用 git log 確認提交紀錄
{% highlight Shell %}
$ git log
# commit d9f0b6880081ba202e27feeaa1f96558354eb4f0 (HEAD -> master)
# Author: Lin Chieh <jaycetyle@gmail.com>
# Date:   Mon Feb 19 19:23:38 2018 +0800

#     Add README
{% endhighlight %}

　

## 一次 git add 多個檔案
　　在改程式的時候經常會有多個檔案變更會需要一次提交，一個一個 git add 就太麻煩了，git add 也有提供以下兩種方式讓你一次加入多個檔案

**　1. git add *.c**
:	git add 支援萬用字元 (*)，這個範例是一次 git add 所有 .c 檔

**　2. git add -A**
:	一次加入所有變更。這個會是 git add 最常用的用法

　　如果要刪除檔案，可以直接將檔案刪除後，用 git add 將刪除檔案這項**變更**加入到暫存區，之後再提交。如果要重新命名檔案也是相同的方式，但要刪除和重新命名的檔案都需要 git add 過才可以。

　

## git commit 的一些常用變化
　　git commit 也有許多可選參數，以下列出幾個常用的

**　1. git commit \-a**
:	非常好用的指令。如果變更都只是修改檔案的話 (刪除、新增檔案不算)，可以使用這個方式省略 git add 的步驟，直接提交變更

**　2. git commit \-\-amend**
:	將變更合併到上次提交中，也是非常好用的選項。有時候一些臨時又做了一些小修改不想要再提交一個新的，可以使用這個方式合併到上一個提交之中，這個指令也可以用來修改上一次的 commit message

**　3. git commit \-s**
:	再提交訊息最後面加上一個 signoff by，簽個名的意思，某些開源的社群複審時會加上 signoff，例如 Linux Kernel

**　4. git commit \-\-allow-empty**
:	這個比較少用但也不是完全無用，只是想要在 git log 上留一些訊息時可以使用此方式，常見的使用情境是執行某些動作，例如版本合併，團隊有某個固定的流程，但到特定步驟時恰好沒有任何變更，可以留個 message 提醒只是剛好沒有變更，不是步驟漏掉。