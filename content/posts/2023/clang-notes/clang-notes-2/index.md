---
title: C Programming 入门札记 02 - 结构体与联合体
description: Struct、Union 以及 typedef 命令
date: 2023-09-01
draft: true
slug: clang-notes-2
categories:
    - Dev
tags:
    - Clang 
image: head.png
---

## 结构体

显然，C 没有类和对象。

```c
struct fraction {
    int numerator;
    int denominator;
};
```
要注意 struct 的末尾加分号，这是常见错误。

用法示例：

```c
struct fraction f1;

f1.numerator = 22;
f1.denominator = 7;
```
比较特殊的是 C 结构声明变量需要有`struct`关键字。

用大括号一次性赋值：

```c
struct car {
    char* name;
    float price;
    int speed;
};

struct car saturn = {"Saturn SL/2", 16000.99, 175};
```
注意，大括号里面的值的顺序，必须与 struct 类型声明时属性的顺序一致。

否则，必须为每个值指定属性名:

```c
struct car saturn = {.speed=172, .name="Saturn SL/2"};
```

没赋值的属性会被默认初始化为`0`。

struct 的数据类型声明语句与变量的声明语句，可以合并为一个语句：

```c
struct {
    char title[500];
    char author[100];
    float value;
} b1 = {"Harry Potter", "J. K. Rowling", 10.0},
  b2 = {"Cancer Ward", "Aleksandr Solzhenitsyn", 7.85};
```
指针变量也可以指向 struct 结构：

```c
struct foo {
    int a;
    int b;
    char* c;
};

struct foo* f1;
```

struct 结构也可以作为数组成员：

```c
struct fraction numbers[1000];
```

struct 结构占用的存储空间，不是各个属性存储空间的总和，而是**最大内存占用属性**的存储空间的**倍数**，其他属性会添加空位与之对齐：

```c
struct foo {
    int a;        // 4
    char pad1[4]; // 填充4字节
    char *b;      // 8
    char c;       // 1
    char pad2[7]; // 填充7字节
};

printf("%d\n", sizeof(struct foo)); // 24
```

这样可以提升执行效率。

## struct 的复制

当一个 struct 变量用等号赋值给另一个变量时，会产生一个**副本**。

这一点跟数组完全不同，使用赋值运算符复制数组，不会复制数据，只会共享地址。

另外，C 语言没有提供比较两个自定义数据结构是否相等的方法，无法用比较运算符（比如`==`和`!=`）比较两个数据结构是否相等或不等。

## struct 指针

如果将 struct 变量传入函数，函数内部得到的是一个原始值的副本。

想要传入同一份数据而非副本，要使用指针：

```c
void happy(struct turtle* t) {
    (*t).age = (*t).age + 1;
}

happy(&myTurtle);
```
上段代码的`(*t)`写法是因为`.`运算符的优先级高于`*`运算符。

`(*t).age`这样的写法很麻烦。C 语言就引入了一个新的箭头运算符（`->`）直接获取属性：

```c
void happy(struct turtle* t) {
    t->age = t->age + 1;
}
```
## struct 的嵌套

```c
struct species {
    char* name;
    int kinds;
};

struct fish {
    char* name;
    int age;
    struct species breed;
}   
```
赋值有多种方法：

```c
// 写法一
struct fish shark = {"shark", 9, {"Selachimorpha", 500}};

// 写法二
struct species myBreed = {"Selachimorpha", 500};
struct fish shark = {"shark", 9, myBreed};

// 写法三
struct fish shark = {
    .name="shark",
    .age=9,
    .breed={"Selachimorpha", 500}
};

// 写法四
struct fish shark = {
    .name="shark",
    .age=9,
    .breed.name="Selachimorpha",
    .breed.kinds=500
};

printf("Shark's species is %s", shark.breed.name);
```
另一个嵌套的例子：

```c
struct name {
    char first[50];
    char last[50];
};

struct student {
    struct name name;
    int age;
} sdt;

strcpy(sdt.name.first, "Harry");
strcpy(sdt.name.last, "Potter");

// 或者
struct name myname = {"Harry", "Potter"};
student1.name = myname;
```
对字符数组属性赋值，要使用`strcpy()`函数，不能直接赋值，因为直接改掉字符数组名的地址会报错。

现在，咱们来实现一个简单**链表**：

```c
struct node {
    int data;
    struct node* next;
};

struct node* head;

// 生成一个三个节点的列表 (11)->(22)->(33)
head = malloc(sizeof(struct node));

head->data = 11;
head->next = malloc(sizeof(struct node));

head->next->data = 22;
head->next->next = malloc(sizeof(struct node));

head->next->next->data = 33;
head->next->next->next = NULL;

// 遍历这个列表
for (struct node *cur = head; cur != NULL; cur = cur->next) {
    printf("%d\n", cur->data);
}
```
## 弹性数组成员

可以用 struct 来编写弹性数组：

```c
struct vstring {
    int len;
    char chars[];
};
```
为这个结构体分配内存：

```c
struct vstring* str = malloc(sizeof(struct vstring) + n * sizeof(char))

str->len = n; // 用 len 属性来记录 n 值
```

## union 联合体

与 struct 的区别是：联合体（union）成员不能同时占用内存空间，长度等于**最长成员的长度**，可以大大节省内存空间。

```c
union student {
    int age;
    char* name;
};
```

## typedef 命令

显然，就是用来起别名的：

```c
typedef int* intptr;

int a = 10;
intptr x = &a;
```