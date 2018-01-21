---
layout: post
title: 在 Linux 寫程式 - gcc 及 Makefile 教學筆記 (2)
description: "gcc 及 Makefile 教學筆記 (2)"
modified: 2015-01-23
tags: [初心者, C/C++]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

### 實驗三 : 複雜一點的 Makefile

　　根據之前提到的規則，我們可以讓 Makefile 再複雜一點，如下:
{% highlight cmake %}
test: main.o op.o
	cc main.o op.o -o test
main.o: main.c
	cc -c main.c
op.o: op.c
{% endhighlight %}

<!--more-->

　　這個範例就比較有趣一點，先看到 test 的命令部分，原本的 gcc 變成了 cc，這是一個替代用法， cc 是 Unix 的編譯器，在 Linux 環境下預設會連結到 gcc，這主要是為了程式碼的移植性，讓 Unix 的程式碼可以在不用更改的情況下由 gcc 編譯，但目前這種需求應該是比較少了。接著，這個範例把產生執行檔的方式拆成了兩個步驟，先從 .c 檔生成 .o，再從 .o 連結成執行檔。make 在執行的時候會先尋找第一個 target 作為他的最終目標，這份 Makefile 的第一個 target 就是 test，根據相依性，他需要 main.o 及 op.o，於是開始往下尋找如何生成這兩個檔案，而生成方式我們已經寫在後面了。

　　另外，這個範例有個不協調的地方，就是 main.o 的部分有命令，而 op.o 的部分沒有？這不是忘記寫，而是使用了自動推導的功能。由於生成目的檔的方式很單純，只要找到同名的 .c 檔並使用 gcc -c 的命令就可以產生了，所以如果沒有特殊的需求， make 可以自行推導生成方式，從這個思維向上延伸，實際上連 op.o: op.c 都可以不用寫，因為看到 op.o 自然也猜得到他的相依檔案會是 op.c ，所以這個 Makefile 還可以再濃縮成 :

{% highlight cmake %}
test: main.o op.o
	cc main.o op.o -o test
{% endhighlight %}

　　和實驗二相比，這種 make 的方式一定會產生目的檔(.o)。一般而言會比較喜歡有產生目的檔的方式，因為如果我們有多個步驟會需要去連結各個目的檔的時候，目的檔的存在可以減少重複編譯的次數。

　

### 實驗四 : 在 Makefile 裡加入一點變數

　　雖然我們的範例程式很簡單，但我們還是可以來測試一些更複雜的 Makefile 方式。現在我們在資料夾內再創立一個名為 include 的資料夾，並把 op.h 放進該資料夾內，來模擬標頭檔在其他路徑的情況，接著把 Makefile 改成以下內容 :
{% highlight cmake %}
CC = gcc
INC = -I ./include

test: main.o op.o
	$(CC) -o $@ main.o op.o
%.o: %.c
	${CC} $< ${INC} -c
{% endhighlight %}

　　這個範例看起來恐怖多了，但仔細分析後會發現也沒有這麼難。首先，我們在一開始使用 = 符號宣告了兩個變數，第一個是 CC ，代表我們所要採用的編譯器形式，實際上，這個變數同時也是一個內建變數，如果我們沒有特別指定，他所代表的就是 cc。第二個變數 INC 則用來表示標頭檔引入路徑，看到前面的 -I 是 gcc 指定引入路徑的參數，而後面就是引入的路徑。

　　$(var) 就是使用變數的方式了，雖然說是變數，但他的行為其實和 C 語言內的 #define macro 很像，可以把他想成是純粹文字的替代。另外，這裡還看到兩個比較特別的變數 $@ 及 $<。 $@ 指的就是該 command 所屬的 target，所以範例中的 $@ 代表的就是 test ，而 $< 指的是則第一個相依檔案。

　　現在會有疑問了， %.o 和 %.c 又是什麼？其實這是一個樣板規則，很像 C++ 的 template，可以替代成符合規則的文字，例如，當 make 在嘗試建立 test 時，會需要 main.o，如果找不到 main.o，make 會開始向下尋找產生 main.o 的方法，此時的 % 就變成了 main，%.o 就成了 main.o，而 %.c 就是 main.c。

　　到這裡可能還有點搞不懂，我們可以直接執行 make 看看到底做了些什麼，再比對一下先前的 Makefile ，馬上就一目了然 :
{% highlight cmake %}
gcc main.c -I ./include -c
gcc op.c -I ./include -c
gcc -o test main.o op.o
{% endhighlight %}

　

### 實驗五 : 清理編譯產生的檔案

　　編譯後一定會產生檔案，有執行檔、目的檔等等。有時我們會希望資料夾可以回到只有 source code 的乾淨狀態，這時就需要把這些編譯產生物給清除掉，這個動作我們也可以請 Makefile 來協助處理，參考以下的 Makefile :
{% highlight cmake %}
CC = gcc
INC = -I ./include
OBJ = main.o op.o

test: ${OBJ}
	$(CC) -o $@ ${OBJ}
%.o: %.c
	${CC} $< ${INC} -c

.PHONY: clean
clean:
	rm -f test ${OBJ}
{% endhighlight %}

　　完成以上的 Makefile 後，在命令列輸入 make clean 便可以執行清理的工作，不必手工再刪除檔案了，可喜可賀，我們立刻來解析這個 Makefile，看看是怎麼做的：

　　首先，在實驗四的時候我們發現  "main.o op.o" 重複寫了兩次，為了避免重複程式碼，我們把它精煉成變數，同時也方便未來擴充。接著我們加上 clean 這個 target，負責清除的工作，你也可以取別的名稱，但 clean 是一個習慣的用法。clean 方法就是用 rm 這個指令，由於移除的目標有可能不存在，為了避免 rm 在刪除不存在的檔案時報錯，我們加上 -f 參數。

　　我們還有一個沒看過的東西 : .PHONY。這個符號的目的是告訴 make，"clean" 不是一個真正的檔案目標，只是一個標記，不要把他當成檔案來處理，避免有檔案真的叫 clean 時，make 會在依賴性判斷時判斷錯
誤，那就糗了。

---
　　到這裡，我們的 Makefile 可自動幫我們執行一連串的編譯指令、檢查檔案相依性關係來決定是否要編譯，以減少編譯時間，同時我們也寫了 clean 這個 target 來替我們做清除的工作，已經是一個相當完整的 Makefile。

　　我第一次看 Makefile 時也覺得好多符號好複雜，稍微研究了一下就比較看得懂了，會發現其實也沒那麼困難，所以把學習的過程整理成這篇，希望可以幫助剛進入 Linux 世界寫程式的朋友們。