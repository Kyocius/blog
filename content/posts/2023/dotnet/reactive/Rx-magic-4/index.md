---
title: Rx.NET 响应式编程指北 04 - 异步与可观察流
date: 2022-08-12
draft: true
slug: rx-magic-4
categories:
    - Dev
tags:
    - Rx.NET
    - CSharp
featured_image: head.jpg

---

## 前言

这回我们来试着一步步把可观察流变成异步的。

## 构建

### 同步写法

现在你有一个可以生成素数的类，假设它耗时很长：

```c#
class MagicalPrimeGenerator
{
    public IEnumerable<int> Generate(int amount) {. . .} 
}
```

你如此使用它：

```c#
var generator = new MagicalPrimeGenerator();
foreach (var prime in generator.Generate(5))
{
    Console.Write("{0}, ", prime);
}
```

很明显这是同步（Sync）的代码，会阻塞线程。

我们尝试把它改写成：

```c#
Task<IReadOnlyCollection<int>> GenerateAsync(int amount);
```

可是就我们就必须 `await` 代码，直到素数全部生成完毕，才能得到返回值。

差强人意。有没有什么办法可以异步迭代返回呢？

### 使用 Rx 改写

Rx.NET 可以异步地让**每一个**素数生成完毕就直接返回，而无需等待全部素数生成才返回。

```c#
public IObservable<int> GeneratePrimes(int amount)
```

我们试着实现它：

```c#
public IObservable<int> GeneratePrimes(int amount)
{
    return Observable.Create<int>(o =>
    {
        foreach (var prime in Generate(amount)) 
        {
            o.OnNext(prime); 
        }
        o.OnCompleted(); 
        return Disposable.Empty; 
    });
}
```

用 `TimeStamp` 打印时间戳：

```c#
var generator = new MagicalPrimeGenerator();
var subscription = generator
    .GeneratePrimes(5)
    .Timestamp()
    .SubscribeConsole("primes observable");
Console.WriteLine("Generation is done");
Console.ReadLine(); 
```

输出：

```
primes observable - OnNext(2@01/08/2022 12:50:02 +00:00)
primes observable - OnNext(3@01/08/2022 12:50:04 +00:00)
primes observable - OnNext(5@01/08/2022 12:50:06 +00:00)
primes observable - OnNext(7@01/08/2022 12:50:08 +00:00)
primes observable - OnNext(11@01/08/2022 12:50:10 +00:00)
primes observable - OnCompleted()
Generation is done
```

接下来我们来修复订阅阻塞的问题。

> 注意，在 Create 方法里套 Task 是糟糕的设计模式，在这里使用主要是为了代码便于理解。我们将在第九章更深入地探讨。

目前为止，我们的可观察流都是在订阅时立即发射数据，观察者没有机会取消订阅。为了改进，我们要使用 `CancellationToken` 
