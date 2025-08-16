---
title: "笨方法学 C 附加题不完全解答"
date: 2024-01-23
draft: false
slug: clang-review
description: Partial Answers for Learn C The Hard Way
categories:
    - Dev
tags:
    - 一生一芯
    - Clang
---
# 前言

笔者撰写本文时还在上大一，数据结构基础几乎为零，所以会有一些可笑的问题和不合时宜的吐槽。

本文尝试解答书后附加题，有的章节太简单就略过了。

# 练习10：字符串数组和循环

1. 如何使用 `,`（逗号）字符来在for循环的每一部分中，`;`（分号）之间分隔多条语句？

```c
for (i = 0, j = 10; i < 10; i++, j--)
```

2. 查询 `NULL` 是什么东西，看看它会打印出什么？

在 C 语言中，NULL 是一个宏定义，通常用于表示指针不指向任何有效的对象或地址。NULL 的确切定义可能因编译器和平台而异，但通常它被定义为 `(void *)0`，即一个转换为 `void` 指针类型的零值。

当你尝试打印一个指向 `NULL`的指针时，结果会取决于你是如何打印它的。在 C 语言中，如果你使用 `%p` 格式化标志（用于打印指针地址）和 `printf` 函数，通常会打印出一个表示空指针的值，通常是 `(nil)` 或者 `0x0`，具体取决于编译器和平台。

3. 看看你是否能在打印之前将 `states` 的一个元素赋值给 `argv` 中的元素，再试试相反的操作？

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
  
  
    char *states[] = {"California", "Oregon", "Washington", "Texas"};

    // 将states的一个元素赋值给argv的一个元素
    // 注意：这通常不是个好主意，因为argv通常不应被修改
    if (argc > 1) {
        argv[1] = states[0];
        printf("argv[1] is now %s\n", argv[1]);
    }

    // 将argv的一个元素赋值给states的一个元素
    states[0] = argv[0];
    printf("states[0] is now %s\n", states[0]);

    return 0;
}
```

# 练习16：结构体和指向它们的指针

1. 如何在栈上创建结构体，就像你创建任何其它变量那样？
2. 如何使用 `x.y` 而不是 `x->y` 来初始化结构体？

对于前两个问题其实非常好解答。只要把变量 `x` 设置成非指针即可。

3. 如何不使用指针来将结构体传给其它函数？

C 语言**不可能**不使用指针传递结构体本身。所以这道题的意思就浅显地指，把参数的 `*` 去掉你会不会...

详细代码请见：[GitHub](https://github.com/eleloya23/Learn-C-The-Hard-Way/blob/master/Ex16/ex16_e.c)

# 练习17：堆和栈的内存分配

> 对于现在你们这些年轻人来说，编程简直太容易了。如果你玩玩 Ruby 或者 Python 的话，只要创建对象或变量就好了，不用管它们存放在哪里。

理清内存最简单原则：如果你的变量并不是从 `malloc` 中获取的，也不是从一个从 `malloc` 获取的函数中获取的，那么它在栈上。

1. `strncpy` 有什么设计缺陷？

```c
char *strncpy(char *dest, const char *src, size_t n)
```

`strcpy` 函数不会向 `dest` 追加 `\0`，也就是字符串没有了结束，可能会造成内存非法访问。

2. C 如何打包结构体？结构体添加一些字段之后的新大小？

在 C 语言中，结构体的默认对齐方式通常是按照结构体成员中占用内存最大的数据类型进行对齐。这被称为“最大成员对齐”或“自然对齐”。

结构体的总大小是其最大对齐成员的大小的整数倍。

# 练习18：函数指针

函数指针编写窍门：

- 编写一个普通的函数声明：`int callme(int a, int b)`
- 将函数用指针语法包装：`int (*callme)(int a, int b)`
- 将名称改成指针名称：`int (*compare_cb)(int a, int b)`

我们可以使用 `typedef` 来简化函数指针以便于传参：

```c
typedef int (*compare_cb)(int a, int b)
```

如此，在另一个函数中，我们就能将 `compare_cb` 作为类型传递：

```c
int *bubble_sort(int *numbers, int count, compare_cb cmp)
```

1. 将错误的函数传给 `compare_cb`，并看看 C 编辑器会报告什么错误？

类似于 "assignment from incompatible pointer type" 或 "incompatible types in assignment" 的错误消息。

2. 将 `NULL` 传给它，看看程序中会发生什么。然后运行 Valgrind 来看看它会报告什么？

```c
#include <stdio.h>

void myFunction(int *ptr) {
    if (ptr != NULL) {
        // 在这里解引用非空指针
        *ptr = 42;
    }
}

int main() {
    int *nullPointer = NULL;

    // 将 NULL 传递给函数指针
    myFunction(nullPointer);

    return 0;
}
```

Valgrind 的报错：

```
==12345== Invalid write of size 4
==12345==    at 0x80484A7: myFunction (example.c:6)
==12345==    by 0x80484F4: main (example.c:14)
==12345==  Address 0x0 is not stack'd, malloc'd or (recently) free'd
```
这个报告表明在 `myFunction` 函数的第 6 行发生了一个尝试写入大小为 4 的无效内存。

# 练习32：双向链表

强烈建议 0 数据结构基础先去油管看一下印度老哥的双向链表教程：[YouTube](https://www.youtube.com/watch?v=nsMWbs-r4vE&list=PLBlnK6fEyqRg7pacSDMgPn7vDVhz3N1uf&index=2)

先在头文件中编写两个结构体：

```c
#include <stdlib.h>

struct ListNode;

typedef struct ListNode
{
    struct ListNode *prev;
    struct ListNode *next;
    void *value; // 可以指向任何数据类型
} ListNode;

typedef struct List
{
    ListNode *first;
    ListNode *last;
    int count;
}
```
然后预定义四个 List 的整体方法：
```c
List *List_create();
void List_destroy(List *list);
void List_clear(List *list);
void List_clear_destroy(List *list);
```
三个宏方法（不知道为什么要用宏）：

```c
#define List_count(A) ((A)->count)

#define List_first(A) ((A)->first != NULL ? (A)->first->value : NULL)

#define List_last(A) ((A)->last != NULL ? (A)->last->value : NULL)
```
接下来预定义数据的操作方法：
```c
void List_push(List *list, void *value);
void *List_pop(List *list);

// 链表的头部插入与删除，命名风格来自于 Perl 语言
void List_unshift(List *list, void *value);
void *List_shift(List *list);

void *List_remove(List *list, ListNode *node);
```
定义 `LIST_FOREACH` 宏。书中说这是个常见的习语：
```c
#define LIST_FOREACH(L, S, M, V) ListNode *_node = NULL; \
    ListNode *V = NULL;\
    for(V = _node = L -> S; _node != NULL; V = _node = _node -> M)
```
这个宏利用了一个 `for` 循环，其中 `_node` 用于跟踪当前节点，`V` 用于表示当前节点。在每次迭代中，`_node` 被更新为下一个节点，从而实现链表遍历的封装。

现在开始实现头文件的方法。在 `list.c` 中先完成 List 的整体方法：
```c
#include <lcthw/list.h>

List *List_create()
{
    return calloc(1, sizeof(List));
}

void List_destroy(List *list)
{
    LIST_FOREACH(list, first, next, cur)
    {
        if (cur->prev)
            free(cur->prev)
    }
    free(list->last);
    free(list);
}

void List_clear(List *list)
{
    LIST_FOREACH(list, first, next, cur) 
    {
        free(cur->value);
    }
}

void List_clear_destroy(List *list)
{
    List_clear(list);
    List_destroy(list);
}
```
接下来实现数据的操作方法：
```c
void List_push(List *list, void *value)
{
    ListNode *node = calloc(1, sizeof(ListNode));
    check_mem(node); // 跳转到 error 标签

    node->value = value;

    if(list->last == NULL)
    {
        list->first = node;
        list->last = node;
    }
    else
    {
        list->last->next = node;
        node->prev = list->last;
        list->last = node;
    }
    list->count++;
error:
    return;
}

void *List_pop(List *list)
{
    ListNode *node = list->last;
    return node != NULL ? List_remove(list, node) : NULL;
}

void List_unshift(List *list, void *value)
{
    ListNode *node = calloc(1, sizeof(ListNode));
    check_mem(node);

    node->value = value;

    if(list->first == NULL) {
        list->first = node;
        list->last = node;
    } else {
        node->next = list->first;
        list->first->prev = node;
        list->first = node;
    }

    list->count++;

error:
    return;
}

void *List_shift(List *list)
{
    ListNode *node = list->first;
    return node != NULL ? List_remove(list, node) : NULL;
}

void *List_remove(List *list, ListNode *node)
{
    void *result = NULL;

    check(list->first && list->last, "List is empty.");
    check(node, "node can't be NULL");

    if(node == list->first && node == list->last) {
        list->first = NULL;
        list->last = NULL;
    } else if(node == list->first) {
        list->first = node->next;
        check(list->first != NULL, "Invalid list, somehow got a first that is NULL.");
        list->first->prev = NULL;
    } else if (node == list->last) {
        list->last = node->prev;
        check(list->last != NULL, "Invalid list, somehow got a next that is NULL.");
        list->last->next = NULL;
    } else {
        ListNode *after = node->next;
        ListNode *before = node->prev;
        after->prev = before;
        before->next = after;
    }

    list->count--;
    result = node->value;
    free(node);

error:
    return result;
}
```
现在我们终于实现了双向链表上的所有操作。闹麻了，书上的实现方法感觉跟喂了 Poops 一样。

1. 研究双向和单向链表，以及什么情况下其中一种优于另一种。

单向链表内存小，性能好；双向链表功能强，比如逆序遍历。

2. 研究双向链表的限制。例如，虽然它们对于插入和删除元素很高效，但是对于变量元素比较慢。

双向链表更改一个节点意味着要修改两端的节点，这太地狱了。