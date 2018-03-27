---
layout: post
title: Python Format String 字串格式化整理
description: "Python 從 2.6 開始新增了 .format() 的字串格式化輸出函數，本篇筆記了數值格式化、對齊及時間表示輸出等等範例"
modified: 2018-03-27
tags: [小法師, Python]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　Python 從 2.6 開始新增了 .format() 的字串格式化輸出函數，本篇筆記了各種輸出的範例

　　

<!--more-->

## 基本操作
　.format() 主要搭配 {} 作為替代符進行輸出，{} 的位置會被後方 format 所帶的參數所取代。預設會依照順序輸出，但也可以用數字指定位置、或者是直接給予名稱做指定

　不指定位置，依序輸出
{% highlight python %}
print("{} {} {}".format("A", "B", "C"))
# A B C
{% endhighlight %}

　用數字指定位置
{% highlight python %}
print("{2} {0} {1}".format("A", "B", "C"))
# C A B
{% endhighlight %}

　用名稱指定位置
{% highlight python %}
print("{a} {c} {b}".format((a="A", b="B", c="C")))
# A C B
{% endhighlight %}
　　
## 數值格式化及對齊
　格式化的方式是在後方加上":"及"格式符"，可用的格式符如下

<figure class="highlight" style="background-color:#E0E0FF; color:#303030">
<pre style="font-size:0.8rem; line-height:1.4">
{:d}        : 整數
{:f}        : 浮點數
{:e} {:E}   : 科學記號，例如 1.020000e+01，大小寫就代表 "e" 顯示的大小寫
{:x} {:X}   : 十六進位，大小寫分別表示 A ~ F 要顯示的大小寫
{:o}        : 八進位
{:b}        : 二進位
{:%}        : 以百分比的方式輸出
</pre>
</figure>

　另外可以使用 > ^ < 這三種符號以及數值來搭配佔位及對齊。注意: 數值型別預設是靠右對齊，字串型別預設是靠左對齊
<figure class="highlight" style="background-color:#E0E0FF; color:#303030">
<pre style="font-size:0.8rem; line-height:1.4">
{:>8d}      : 整數靠右對齊，寬度為 8
{:^8d}      : 整數置中對齊，寬度為 8
{:<8d}      : 整數靠右對齊，寬度為 8
{:8.3f}     : 小數點後保留 3 位，總寬度為 8 (含小數點)
{:+8.3f}    : 小數點後保留 3 位，帶正負號，總寬度為 8 (含小數點及正負號)
</pre>
</figure>

　**: 前方的數值如基本操作，代表後方參數的位置**
{% highlight python %}
print "|{0:8d}||{1:<8d}|".format(123, 456)
# |     123||456     |
print("|{0:+8.2f}|".format(4.32))
# |   +4.32|
print("|{0:8.2%}|".format(0.8))
# |  80.00%|
{% endhighlight %}

　**字串與數值輸出預設對齊方式的比較**
{% highlight python %}
print("|{0:8}|".format(123))
# |     123|
print("|{0:8}|".format("abc"))
# |abc     |
{% endhighlight %}
　　
## 時間表示
　datetime 有實作 __format__ 函數，因此也支援 format 輸出，要再加上 % 格式符，列舉如下
<figure class="highlight" style="background-color:#E0E0FF; color:#303030">
<pre style="font-size:0.8rem; line-height:1.4">
%a  : 星期縮寫 *　　　　　(Mon)
%A  : 星期全寫 *　　　　　(Monday)
%w  : 星期數字  　　　　　(0 ... 6)
%d  : 日期  　　　　　　　(01 ... 31)
%b  : 月份縮寫 *　　　　　(Jan)
%B  : 月份全寫 *　　　　　(January)
%m  : 月份數字  　　　　　(01 ... 12)
%y  : 西元年二位數字  　　(00 ... 99)
%Y  : 西元年  　　　　　　(1970 ... 2018)
%H  : 24小時制  　　　　　(00 ... 23)
%I  : 12小時制  　　　　　(01 ... 12)
%p  : AM 或 PM *
%M  : 分  　　　　　　　　(00 ... 59)
%S  : 秒  　　　　　　　　(00 ... 59)
%f  : 微秒，六位數字  　　(000000 ... 999999)
%c  : 日期和時間 *　　　　(Tue Aug 16 21:30:00 2017)
%x  : 日期 *　　　　　　　(08/16/2017)
%X  : 時間 *　　　　　　　(21:30:00)
</pre>
</figure>

　其中有表示 * 根據地區會有不同的輸出，[詳細可參考這裡](https://docs.python.org/2/library/datetime.html)，範例如下

{% highlight python %}
from datetime import datetime
import locale

time = datetime(2018, 1, 23, 14, 12, 34, 123456)
print ("{:%Y-%m-%d %H:%M:%S %A}".format(time))
# 2018-01-23 14:12:34 Tuesday

# 注意: locale 和環境相關，如果環境沒有該地區設定，會出現類似以下的錯誤
# locale.Error: unsupported locale setting
locale.setlocale(locale.LC_ALL, "zh_TW.utf8")
print ("{:%Y-%m-%d %H:%M:%S %A}".format(time))
# 2018-01-23 14:12:34 週二
{% endhighlight %}