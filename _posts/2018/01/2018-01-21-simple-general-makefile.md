---
layout: post
title: 寫一個簡單、通用的 Makefile
description: "簡單通用的 Makefile 筆記"
modified: 2018-01-21
tags: [小法師, C/C++]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　用 GCC 想要 Build Code 的話，除了直接使用 gcc 以外，再來就是寫 Makefile 了。不過很多時候只是想要寫一個簡單的測試小程式或小工具，還要重新寫 Makefile 那就顯得相當麻煩。這篇筆記了一個簡單的通用 Makefile，只需要修改少數幾個地方就可以應付規模不大的程式。

<!--more-->

### 範例
{% highlight cmake %}
TARGET  = helloworld
SRC_DIR = ./
INC_DIR = ./

CFLAGS += -I$(INC_DIR)
SRC     = $(wildcard $(SRC_DIR)*.c)
OBJ     = $(SRC:%.c=%.o) #$(patsubst %.c, %.o, $(SRC))
DEP     = $(SRC:%.c=%.d) #$(patsubst %.o, %.d, $(OBJ))

.PHONY: clean

$(TARGET) : $(OBJ)
    $(CC) $(CFLAGS) $^ -o $@

# Include all .d files
-include $(DEP)

# Build target for every single object file.
# The potential dependency on header files is covered by calling `-include $(DEP)`.
%.o : %.c
    $(CC) $(CFLAGS) -MMD -c $< -o $@

## clean
clean:
    rm -f $(TARGET) $(OBJ) $(DEP)
{% endhighlight %}

### 簡要說明
　　這個　Makefile 會抓取 $SRC_DIR 內的所有 .c 檔，Build 出 .o 並連結成 $TARGET。如果有額外的 include 路徑的話，可以在 $INC_DIR 內做指定。一般的小型程式很夠用囉！
