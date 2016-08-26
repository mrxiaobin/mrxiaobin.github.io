---
layout: post
title: C语言可变参数
date: 2016-8-26
categories: blog
tags: [学习笔记,iOS,C]
description: 
---

## 要用到的定义和宏

 - va_list 
 - va_start 
 - va_arg 
 - va_end

 具体用法参照下边代码实例

## 代码实例

```C
void myprintStr(char *str, ...) {
    va_list arguments;
    va_start(arguments, str);
    char *currentStr = str;
    while (currentStr) {
        printf("%s ", currentStr);
        currentStr = va_arg(arguments, char *);
    }
    printf("\n");
    va_end(arguments);
}

int main(int argc, const char * argv[]) {
    myprintStr("hello", "world", "of", "IT");

    return 0;
}

//hello world of IT
```
