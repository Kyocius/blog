---
title: C Programming 入门札记 04 - 文件操作
date: 2023-09-07
draft: true
slug: clang-notes-4
categories:
    - Dev
tags:
    - Clang 
image: head.png
---

## 文件指针

操作文件时，需要先声明一个文件指针：

```c
FILE* fp;
```
下面是一个读取文件的实例：

```c
#include <stdio.h>

int main(void) {
    FILE* fp;
    char c;

    fp = fopen("hello.txt", "r");
    if (fp == NULL) {
        return -1;
    }

    c = fgetc(fp);
    printf("%c\n", c);

    fclose(fp);

    return 0;
}
```
## 标准流

Linux 系统默认提供三个已经打开的文件，它们的文件指针如下。

- `stdin`（标准输入）：默认来源为键盘，文件指针编号为`0`。
- `stdout`（标准输出）：默认目的地为显示器，文件指针编号为`1`。
- `stderr`（标准错误）：默认目的地为显示器，文件指针编号为`2`。

Linux 系统的文件，不一定是数据文件，也可以是设备文件，即文件代表一个可以读或写的设备。