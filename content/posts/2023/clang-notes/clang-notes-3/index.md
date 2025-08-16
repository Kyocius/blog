---
title: C Programming 入门札记 03 - 预处理器
date: 2023-09-03
draft: true
slug: clang-notes-3
categories:
    - Dev
tags:
    - Clang 
image: head.png
---

## #define

用来将指定的词替换成另一个词。每条替换规则，称为一个**宏**（macro）:

```c
#define MAX 100
```
同名的宏可以重复定义，只要定义相同就没有问题。如果定义不同，就会报错。

## 带参数的宏

### 基本用法

长得像函数，但不完全是函数：

```c
#define SQUARE(X) X*X

// 输出19
printf("%d\n", SQUARE(3 + 4));
```
`SQUARE(3 + 4)`如果是函数，输出的本应该是 49（`7*7`）；

但因为宏是原样替换，`3 + 4*3 + 4` = 19。

宏的优点是相对简单，本质上是字符串替换，不涉及数据类型，不像函数必须定义数据类型。性能也会更好一些。

### `##`运算符

```c
#define XNAME(n) "x"#n

// 输出 x4
printf("%s\n", XNAME(4));
```
上述代码`"x"`指定输出的格式是字符串。

### 不定参数的宏

```c
#define X(a, b, ...) (10*(a) + 20*(b)), __VA_ARGS__
```
## #undef

用来取消已经使用`#define`定义的宏。

GCC 的`-U`选项可以在命令行取消宏的定义。

## #include

用于加载文件：

```c
// 形式一
#include <foo.h> // 加载系统提供的文件

// 形式二
#include "foo.h" // 加载用户提供的文件
```
形式一，编译器会到系统指定的安装目录里面，去寻找这些文件。

形式二，可以自己指定：

```c
#include "/usr/local/lib/bin/foo.h"
```
## #if...#endif

#if的常见应用就是打开（或关闭）调试模式:

```c
#define DEBUG 1

#if DEBUG
printf("value of i : %d\n", i);
printf("value of j : %d\n", j);
#endif
```
GCC 的`-D`参数可以在编译时指定宏的值，可以方便地打开调试：

```c
$ gcc -DDEBUG=1 foo.c
```
## #ifdef...#endif

一般是用来检查是否重复引入某个库：

```c
#define EXTRA_HAPPY

#ifdef EXTRA_HAPPY
  printf("I'm extra happy!\n");
#endif
```
## #pragma

用来修改编译器属性:

```c
// 使用 C99 标准
#pragma c9x on
```
上面示例让编译器以 C99 标准进行编译。
