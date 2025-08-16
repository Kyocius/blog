---
title: Rx.NET 响应式编程指北 01 - 基础概念
date: 2022-08-07
description: Rx.NET in Action 笔记
draft: false
slug: rx-magic-1
categories:
    - Dev
tags:
    - Rx.NET Tutorial
    - CSharp
featured_image: head.jpg

---

## 前言

最近刚读了 *Rx.NET in Action*，学到的新概念非常多，担忧自己会日渐遗忘，觉得有必要写一篇博客记录一下，说不定也能帮到一些志同道合的朋友。

本系列将会使用我深爱的 C# 语言和 Rx.NET 框架来讲解响应式编程（Reactive Programming）的魔法 ╰(*°▽°*)╯

## 背景知识

- C# 中的**委托**、**泛型**、**事件**以及 **LINQ** 等概念
- Rx = Reactive Extension
- Rx 是由**微软**旗下一个实验室发起的  ~~巨硬大法好~~

## 响应式概念

Rx 到底是干啥的？

首先来看看微软的解释：Rx = Observables + LINQ + Schedulers。谔谔，啥玩意儿？

那看看大佬怎么解释的吧：Rx = 处理异步数据流。这个解释还行，但还是不明白。

这里是更通俗的解释：

```c#
var a = 1;
var b = 2;
var c = a + b; // c 的值显然为 3
```

当你改变 `a` 或者 `b` 值的时候，按照我们以前的思维，`c` 的值并不会随之发生变化。

如果我们想要 `c` 的值随之而改变呢？很简单，快快使用 **Rx**！

### 可观察对象 Observable

```c#
public interface IObservable<T>
{
    IDisposable Subscribe(IObserver<T> observer);
}
```

- 接口只有一个方法 `Subscribe`，返回 `IDisposable` 对象（代表着订阅事件本身）
- `Observable` 持有 `Observer` 的集合，在值改变时随时通知它们
- `IDisposable` 对象可以通过  `Dispose` 方法随时取消订阅

### 观察者 Observer

观察者的源码其实也很简单。

```c#
public interface IObserver<T>
{
    void OnNext(T value); 
    void OnError(Exception error); 
    void OnCompleted(); 
}
```

- `OnNext` 方法定义当观测的值出现变化时，该做什么（类比迭代）
- `OnError` 方法定义它出现错误时该做什么
- `OnCompleted` 方法定义它任务完成时该做什么
- 值得注意的是：发生错误时，数据流就会中断

### 操作符 Operator

Rx 带来了海量的操作符，帮助我们分类、查找、转换数据。

Rx.NET 的操作符与其它语言的 Rx 不同的是，操作符采用了原汁原味的 LINQ 风格。

下面来点简单实例：

```c#
IObservable<string> strings= ... // 数据源

IDisposable subscription = strings // 一个 IDisposable 代表了订阅事件本身
    .Where(str => str.StartsWith("A"))
    .Select(str => str.ToUpper())
    .Subscribe(...); // 传入 Observer 或直接传 OnNext 方法

subscription.Dispose(); // 上面说过了，可以随时取消订阅
```

上面的代码读起来是不是很流畅？

其实第一行省略了如何创建可观察数据源，专注于操作符本身，要不然太劝退了。

如何创建可观察的数据源将在后面的章节讲解。

## 理解事件和流 Events & Streams

事件非常好理解，比如我们点击一个按钮就是一个事件。

在 C# 中，事件由**数据源**「Event Source」和**处理器**「Event Handler」组成。

在 Rx 中，「Event Source」对应「Observable」而「Event Handler」对应「Observer」。

### 万物皆流

>  A data stream is like a hose: every drop of water is a data packet that needs to go through stations until it reaches the end. Your data also needs to be filtered and transformed until  it gets to the real handler that does something useful with it.

上面这段 Quote 来自 *Rx.NET in Action*，大致的意思就是数据就像水管子一样，我们通过操作符过滤水流，最后通过喷头将水流传播。

我认为这是响应式的核心概念。只要你想持续地监听某数据的变化，那么使用 Rx 让数据变成可观察流，绝对是个好主意。
