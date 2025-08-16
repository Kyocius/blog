---
title: C Programming 入门札记 01 - 内存管理
date: 2023-08-31
draft: true
slug: clang-notes-1
categories:
    - Dev
tags:
    - Clang 
image: head.png

---

## 前置知识

**栈**是由系统管理的，主要是函数中的局部变量。**堆**是由用户管理的，主要是全局变量。

## void 指针

没有类型，只有内存块的地址信息。需要类型时再向编译器补充。

可以指向任意数据，但不能解读数据。

任意指针可以与 void 互转：

```c
int x = 10;

void* p = &x; // 整数指针转为 void 指针
int* q = p;   // void 指针转为整数指针
```

void 指针很重要，因为很多函数的返回值就是 void 指针。

## malloc()

`malloc()` 函数用于请求一段内存，系统会从 Heap 中分配一段连续的内存给它。

原型定义在头文件 `stdlib.h` 中：

```c
void* malloc(size_t size)
```

常见用法是：

```c
int* p = malloc(sizeof(int));

*p = 12;
printf("%d\n", *p); // 12
```

`malloc()` 函数有可能分配内存失败，返回常量 `Null`。所以在使用它之后要检查：

```c
int* p = malloc(sizeof(int));

if (p == NULL) {
  // 内存分配失败
}

// or
if (!p) {
  //...
}
```

`malloc()` 最常用的场景是，为**数组**和**自定义数据类型**分配内存：

```c
int* p = (int*) malloc(sizeof(int) * 10)

for (int i = 0; i < 10; i++)
    p[i] = i * 5;
```
`malloc()` 用于创建数组的好处是可以创建**动态数组**：

```c
int* p = (int*) malloc(n * sizeof(int));
```

malloc() 不负责把内存初始化，需要自己动手：

```c
char* p = malloc(8);
strcpy(p, "kyocius");
```

## free()

`free()` 用于释放 `malloc()` 分配的内存，原型也在`stdlib.h` 里面：

```c
// block 是 malloc() 返回的内存地址
void free(void* block) 
```

用法示例：

```c
int* p = (int*) malloc(sizeof(int));

*p = 12;
free(p);
```

别忘了每次 `malloc()`，都要用 `free()` 释放内存。

## calloc()

区别一：参数不同。
```c
int* p = calloc(10, sizeof(int));

// 等同于
int* p = malloc(sizeof(int) * 10);
memset(p, 0, sizeof(int) * 10);
```

区别二：`calloc()` 会将所分配的内存全部初始化为 `0`。

## realloc()

用于修改分配的内存大小。

```c
void* realloc(void* block, size_t size)
```

`realloc()` 同样不会对内存进行初始化。

## restrict 限制符

受限指针，不让别的指针读取当前内存。

```c
int* restrict p;
p = malloc(sizeof(int));

int* q = p;
*q = 0;  // 未定义行为
```

如上，想访问内存只能通过变量 `p`。

## memcpy()

用于将一块内存拷贝到另一块内存。原型定义在头文件`string.h`。

```c
void* memcpy(
    void* restrict dest, 
    void* restrict source, 
    size_t n
);
```
上一段的代码的参数`n`要注意。如果要拷贝 10 个 double 类型的数组成员，`n`就等于`10 * sizeof(double)`，而不是`10`。

用法示例：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
  char s[] = "Goats!";
  char t[100];

  memcpy(t, s, sizeof(s));  // 拷贝7个字节，包括终止符

  printf("%s\n", t);        // "Goats!"

  return 0;
}
```
`memcpy()`比`strcpy()`更加安全和快速。前者不检查字符串末尾的`\0`。

## memmove()

```c
int a[100];

memmove(&a[0], &a[1], 99 * sizeof(int));
```

上段代码，让从 `a[1]` 至第 99 个成员，都向前移动一个位置。

下面是另一个例子：

```c
char x[] = "Home Sweet Home";

// 输出 Sweet Home Home
printf("%s\n", (char *) memmove(x, &x[5], 10));
```

从字符串`x`的 5 号位置开始的 10 个字节，就是`Sweet Home`，`memmove()`将其前移到 0 号位置。

## memcmp()

用于比较两份内存区域：

```c
char s1[] = {'b', 'i', 'g', '\0', 'c', 'a', 'r'};
char s2[] = {'b', 'i', 'g', '\0', 'c', 'a', 't'};

if (memcmp(s1, s2, 3) == 0) // true
if (memcmp(s1, s2, 4) == 0) // true
if (memcmp(s1, s2, 7) == 0) // false
```

它的返回值是一个整数（因为 C 没有正经的布尔值）。两块内存区域的每个字节以字符形式解读，按照字典顺序进行比较。

- 如果两者相同，返回 0；
- 如果 s1 大于 s2，返回大于 0 的整数；
- 如果 s1 小于 s2，返回小于 0 的整数。 