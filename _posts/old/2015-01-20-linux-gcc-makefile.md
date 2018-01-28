---
layout: post
title: 在 Linux 寫程式 - gcc 及 Makefile 教學筆記 (1)
description: "gcc 及 Makefile 教學筆記 (1)"
modified: 2015-01-20
tags: [初心者, C/C++]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　剛從 Windows 跳到 Linux 的 Programmer 應該都會跟我遇到類似的問題吧，就是 Linux 到處都是看不懂的 Makefile，畢竟 Windows 沒有這種東西，在 Windows 裡，Makefile 的工作都由 IDE 代勞了，這裡不得不說 Visual Studio 真的是很強大的整合開發環境，會寵壞小孩。但現在要進到 Linux 的領域，就勢必要搞懂這個 Makefile 在做什麼，大概怎麼寫，不然會遇到蠻多障礙，而這篇就是筆記 Makefile 的使用方式。

<!--more-->

　　首先，準備以下三個檔案以方便我們之後的測試 :

**main.c**
{% highlight c++ %}
#include <stdio.h>
#include "op.h"

int main(void)
{
    int a, b;
    printf("Enter two numbers to add\n");
    scanf("%d%d",&a,&b);
    printf("Sum of entered numbers = %d\n", add(a, b));

#ifdef TEST
    printf("Program End\n");
#endif
    return 0;
}
{% endhighlight %}
**op.h**
{% highlight c++ %}
#ifndef OP_H
#define OP_H

int add(int a, int b);

#endif
{% endhighlight %}
**op.c**
{% highlight c++ %}
#include "op.h"

int add(int a, int b)
{
    return a + b;
}
{% endhighlight %}

　　請把三個檔案都放在同一個路徑內。程式很簡單，相信會 C 語言的應該都看得懂。準備好程式以後，接著就要開始來編譯以產生執行檔，要怎麼編呢？

　

### 實驗 (一) 直接使用 gcc

　　直接 gcc 應該是最簡單的辦法，使用以下命令來呼叫 gcc 幫我們編譯 :

　　*$ gcc main.c op.c -o test*

　　gcc 就是我們要使用的編譯器，後面接的兩個檔案名稱就是我們要編譯的檔案名稱，-o 代表我們要指定輸出的執行檔檔名，而後面接的 test 就是指定的檔名。如果不使用 -o 的話，預設的輸出名稱會是 a.out，可以自己試試看。除了 -o 以外，gcc 還有很多編譯器參數可以設定，以下列舉幾個可能會用到的:

1. **-c : 只編譯不連結**  
只產生 obj 檔而不連結成執行檔。  
在這個範例中，可以先把之前產生的 obj 檔和執行檔刪掉，輸入 *$ gcc main.c op.c -c* ，會發現只有 main.o 及 op.o 生成，接著再輸入 *$ gcc main.o op.o -o test* 就可以將兩個 obj 檔連結成執行檔。也就是說，最先前執行的 *$ gcc main.c op.c -o test* 其實是這兩條命令結合而成的。

2. **-S : 只產生組合語言檔案 (.s)**  
這個參數一般情況應該比較少用到，不過可以試試看會生出什麼東西。

3. **-O0, O1, O2, O3 : 啟動編譯最佳化**  
O0 是不最佳化，O3 是最高等級的最佳化。這個東西可以配合上面的 -S 就能看出效果，測試以下兩個編譯命令，可以觀察出有無開最佳化編譯後的差異:  
*gcc op.c -o op.s -S*  
*gcc op.c -o op.s2 -O3 -S*

4. **-I : 指定 include 路徑**  
如果 header 放在其他資料夾的話可以使用此參數，這個東西和 VC project property 中的 Additional Include Directories 類似。

5. **-L : 指定 library 路徑**  
和 -I 類似，只是這裡指定的是 library 的路徑，和 VC  中的 Additional Library Directories 類似。

6. **-Dname : 預定義 macro**  
和 VC 的 Preprocessor Definitions 相同。這個範例可以使用 $ gcc main.c op.c -o -DTEST test ，會多印出 Program End 的文字。

　

### 實驗 (二)  使用 Make

　　在大部分的情況下我們不會直接使用 gcc 命令來編譯，因為如果檔案變多，專案結構複雜，gcc 的命令就會變得很冗長。而 make 其實就像是編譯器的自動腳本，我們把編譯的方法和流程寫在 Makefile 裡面，之後只要輸入 make 命令就可以執行一連串的編譯指令，另外，make 會自動辨識目前程式碼的修改情況，來決定哪些文件要重新編譯，節省編譯時間。

{% highlight cmake %}
test: main.c op.c
	gcc main.c op.c -o test
{% endhighlight %}

　　可以發現內容和最早的編譯指令是一樣的，來簡單說明一下 Makefile 最基本的語法的規則 :
{% highlight cmake %}
target ... : prerequisites ...
	command
	...
{% endhighlight %}

　　**target** 通常是我們最後要產生的目標名字，可能是一個執行檔，或是一個 .o 檔。target 也有可能代表的是一個動作命令，比較常見的有 all 和 clean。

　　**all** 是一個習慣的用法，Makefile 可能有多個編譯流程或步驟，可以選擇只編譯某一項步驟，但如果是 make all 的話，他會執行所有有相依性的編譯步驟以完成一個完整的建置，通常 all 會放在 Makefile 的最前面，也就是預設執行的 target。

　　**clean** 則是執行清除的動作，清除一些編譯過程的中繼檔案或執行檔，當然清除的方式和目標也是要自己寫的。

　　**prerequisites** 是要生成 target 時所相依的檔案，這個範例的話就是兩個 .c 檔，在比較複雜的編譯步驟中，這裡也可以是其他的步驟的 target。在這個範例中這段是可以忽略不寫的，可以試試看。

　　**command** 的部分就是生成 target 所需要執行的命令，而生成的方法就是 gcc 指令了，注意 command 的前面的 tab 是規定的，有縮排可讀性也比較高。

　

### 繼續閱讀
* [在 Linux 寫程式 - gcc 及 Makefile 教學筆記 (2)]({% post_url old/2015-01-23-linux-gcc-makefile-2 %})
* [寫一個簡單、通用的 Makefile]({% post_url 2018/01/2018-01-21-simple-general-makefile %})