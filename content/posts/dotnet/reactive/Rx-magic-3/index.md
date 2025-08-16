---
title: Rx.NET 响应式编程指北 03 - 深入流的创建
description: Rx.NET in Action 笔记
date: 2022-08-10
draft: false
slug: rx-magic-3
categories:
    - Dev
tags:
    - Rx.NET Tutorial
    - CSharp
featured_image: head.jpg

---

## 前言

之前我们见识了 Rx.NET 的基本用法，把 .NET 事件转化成了可观察数据流，并且补充了一些拓展知识（Ex），现在我们要接着深入 Rx.NET 的核心技术了。

## 普通数据流

还记得吗，`IObservable` 接口只有一个方法 `Subscribe`，是 Rx.NET 的基础。

我们将试着打造一个**即时通讯**系统。

### 实现 `IObservable` 接口

在实际的开发中，非常不推荐直接实现 `IObservable` 接口，但是理解这个流程对我们的学习很有帮助。

```c#
using System;
using System.Reactive.Disposables;

public class NumbersObservable : IObservable<int>
{
    private readonly int _amount;

    public NumbersObservable(int amount) 
    {
         _amount = amount;
    }
    public IDisposable Subscribe(IObserver<int> observer)
    {
        for (int i = 0; i < _amount; i++) 
        {
            observer.OnNext(i); 
        } 
        observer.OnCompleted(); 
        return Disposable.Empty; 
    }
}
```

### 实现 `Observer` 接口

```c#
public class ConsoleObserver<T> : IObserver<T> 
{
    private readonly string _name;

    public ConsoleObserver(string name="") 
    {
        _name = name;
    }
    public void OnNext(T value) 
    { 
        Console.WriteLine("{0} - OnNext({1})",_name,value); 
    } 
    public void OnError(Exception error) 
    { 
        Console.WriteLine("{0} - OnError:", _name); 
        Console.WriteLine("\t {0}", error); 
    } 
    public void OnCompleted() 
    { 
        Console.WriteLine("{0} - OnCompleted()", _name); 
    } 
}
```

最后消费它：

```c#
var numbers = new NumbersObservable(5);
var subscription =
    numbers.Subscribe(new ConsoleObserver<int>("numbers"));
```

看看结果：

```
numbers - OnNext(0)
numbers - OnNext(1)
numbers - OnNext(2)
numbers - OnNext(3)
numbers - OnNext(4)
numbers - OnCompleted()
```

我们现在手动实现了一个可观察数据流和观察者。

再来个扩展方法方便我们打印，

注意！这个方法将会**贯穿**之后的章节：

```c#
public static class Extensions {
    public static IDisposable SubscribeConsole<T>(
        this IObservable<T> observable,
        string name = "") {
        return observable.Subscribe(new ConsoleObserver<T>(name));
    }
}
```

### 手动创建 `Observable` 的缺陷

手动创建可观察数据流一般不推荐且很少用，因为如此明显繁琐且容易出错。

除此之外，如果你不小心在 `OnComplete` 方法之后调用了 `OnNext`：

```c#
public IDisposable Subscribe(IObserver<int> observer)
{
    for (int i = 0; i < _amount; i++)
    {
        observer.OnNext(i);
    }
    observer.OnCompleted();

    observer.OnNext(_amount); // 注意这行
    return Disposable.Empty;
}
```

你将得到以下结果：

```
errorTest - OnNext(0)
errorTest - OnNext(1)
errorTest - OnNext(2)
errorTest - OnNext(3)
errorTest - OnNext(4)
errorTest - OnComplete
errorTest - OnNext(5)
```

怎么会事捏， `OnComplete` 之后居然还能订阅到数据？

这是 Rx 背后 `Observable` 与 `Observer` 的 `Repeat` 方法重新订阅导致的。

### `ObservableBase` 接口

手写也不是不可以，比如当你想封装一个复杂的、包含事件属性的接口的时候：

```c#
public interface IChatConnection
{
    event Action<string> Received; 
    event Action Closed; 
    event Action<Exception> Error; 
    void Disconnect(); 
}
```

我们一般会写个 Client：

```c#
public class ChatClient
{
    ...
    public IChatConnection Connect(string user, string password)
    {
       // 连接服务
    }
}
```

再手动改写成一个 `Observable`：

```c#
using System;
using System.Reactive;
using System.Reactive.Disposables;
public class ObservableConnection : ObservableBase<string>
{
    private readonly IChatConnection _chatConnection;
    public ObservableConnection(IChatConnection chatConnection) 
    {
        _chatConnection = chatConnection; 
    } 

    protected override IDisposable SubscribeCore(IObserver<string> observer)
    {
        Action<string> received = message => 
        { 
            observer.OnNext(message); 
        }; 
        Action closed = () => 
        { 
            observer.OnCompleted(); 
        }; 
        Action<Exception> error = ex => 
        { 
            observer.OnError(ex); 
        }; 
        _chatConnection.Received += received;
        _chatConnection.Closed += closed;
        _chatConnection.Error += error;
        return Disposable.Create(() => 
        { 
            _chatConnection.Received -= received; 
            _chatConnection.Closed -= closed; 
            _chatConnection.Error -= error; 
            _chatConnection.Disconnect(); 
        }); 
    }
}
```

> 发现没，这其实就是 SignalR 的写法（微软的后端框架）

现在消费：

```c#
var chatClient = new ChatClient(); 
var connection = chatClient.Connect("guest", "guest"); 
IObservable<string> observableConnection = new ObservableConnection(connection); 

// 最开始的那个类
var subscription = observableConnection.SubscribeConsole("receiver"); 
```

写个扩展方法来简化操作：

```c#
public static class ChatExtensions
{
    public static IObservable<string> ToObservable(this IChatConnection connection)
    {
        return new ObservableConnection(connection);
    }
}
```

综合下来看，这么手动写也太踏马折磨了。Rx 有没有方法来简化这些操作呢？

### `Observable.Create` 方法

别傻乎乎地手写了，Rx 可是提供了好用的工厂函数呢。

```c#
Observable.Create<int>(observer => 
{
    for (int i = 0; i < 5; i++) 
    {
        observer.OnNext(i);
    }
    observer.OnCompleted();
    return Disposable.Empty;
});
```

直接写个 Lambda 表达式来作为 `Subscribe` 方法。

`Observable.Create` 非常灵活，因此也被大量使用在 Rx 的应用中。

### 延迟初始化 Defer

```c#
public IObservable<string> ObserveMessages(string user, string password)
{
    var connection = Connect(user, password); 
    return connection.ToObservable();
}
```

当我们想要这个可观察数据流仅在有观察者订阅它的时候才会被创建时，我们可以使用 `Observable.Defer` 方法来延迟初始化：

```c#
public IObservable<string> ObserveMessagesDeferred(string user, string password)
{
    return Observable.Defer(() =>
    {
        var connection = Connect(user, password);
        return connection.ToObservable();
    });
}
```

需要指出的是，由这些工厂函数创建的可观察数据流并不能共享观察者：

```c#
var messages = chatClient.ObserveMessagesDeferred("user","password");
var subscription1 = messages.SubscribeConsole();
var subscription2 = messages.SubscribeConsole();
```

也就是说， `subscription1` 和 `subscription2` 是两个完全独立的实例。

关于「共享流」「热流」以及「冷流」的知识将会在第六章讲解。

## 由事件转换

在第二章我们已经见识过了，这回来点儿更深入的。

先看定义：

```c#
FromEventPattern<TDelegate, TEventArgs>(Action<TDelegate> addHandler, Action<TDelegate> removeHandler)
```

### 标准事件

来个事件：

```c#
public event RoutedEventHandler Click;
```

这个委托类型是 `System.Windows` 中定义好的：

```c#
public delegate void RoutedEventHandler(object sender, System.Windows.RoutedEventArgs e)
```

转化成可观察的事件流：

```c#
IObservable<EventPattern<RoutedEventArgs>> clicks = Observable.FromEventPattern<RoutedEventHandler, RoutedEventArgs>(
    h => theButton.Click += h,
    h => theButton.Click -= h);

clicks.SubscribeConsole(); 
```

Rx 还提供了一种简化的写法，但并不推荐：

```c#
IObservable<EventPattern<object>> clicks = 
    Observable.FromEventPattern(theButton, "Click");
```

### 不标准事件

并不是所有事件的参数都是一个 `Sender` + 一个 `EventArgs`。

```c#
public delegate void NetworkFoundEventHandler(string ssid);

class WifiScanner
{
    public event NetworkFoundEventHandler NetworkFound = delegate { };
    // ...
}
```

可以看到这个事件需要一个字符串类型的参数。

之前的 `FromEventPattern` 就不能用了，Rx 提供了 `FromEvent`：

```c#
IObservable<TEventArgs> FromEvent<TDelegate, TEventArgs>(Action<TDelegate> addHandler, Action<TDelegate> removeHandler);
```

如此就可以：

```c#
var wifiScanner = new WifiScanner();
IObservable<string> networks = 
    Observable.FromEvent<NetworkFoundEventHandler, string>(
        h => wifiScanner.NetworkFound += h,
        h => wifiScanner.NetworkFound -= h);
```

### 更复杂的事件

Of course，还有更复杂的事件：

```c#
public delegate void ExtendedNetworkFoundEventHandler(string ssid, int strength);

class WifiScanner
{
    public event ExtendedNetworkFoundEventHandler ExtendedNetworkFound = delegate { };
}
```

放心，`FromEvent` 提供了重载：

```c#
IObservable<TEventArgs> FromEvent<TDelegate, TEventArgs>(
    Func<Action<TEventArgs>, TDelegate> conversion,
    Action<TDelegate> addHandler,
    Action<TDelegate> removeHandler);
```

下面需要利用元组（Tuple）的特性：

```c#
IObservable<Tuple<string, int>> networks =
    Observable.FromEvent<ExtendedNetworkFoundEventHandler, Tuple<string, int>>(
        rxHandler => (ssid, strength) => rxHandler(Tuple.Create(ssid, strength)),
        h => wifiScanner.ExtendedNetworkFound += h,
        h => wifiScanner.ExtendedNetworkFound -= h);
```

### 无参事件

```c#
event Action Connected = delegate { };
```

我们需要这么写：

```c#
IObservable<Unit> connected =
    Observable.FromEvent(h => wifiScanner.Connected += h,
                         h => wifiScanner.Connected -= h);

connected.SubscribeConsole("connected");
```

`Unit` 类型代表了「空」，它 `ToString` 之后的值为「()」

所以你会得到以下结果：

```
connected - OnNext(())
connected - OnNext(())
```

## 由迭代类型转换

迭代类型和可观察类型长得很像，Rx 也提供了转换的方法。

### 迭代类型 → 可观察流

```c#
IEnumerable<string> names = new []{"AHpx", "Yoroion", "Weeknic", "GodLeaveMe"};
IObservable<string> observable = names.ToObservable();

observable.SubscribeConsole("names");
```

你将会得到：

```
names - OnNext(AHpx)
names - OnNext(Yoroion)
names - OnNext(Weekenic)
names - OnNext(GodLeaveMe)
names - OnCompleted()
```

如果迭代的过程中发生了错误，那么报错就会传递至 `OnError` 方法：

```c#
class Program
{
    static void Main(string[] args)
    {
        NumbersAndThrow() 
            .ToObservable()
            .SubscribeConsole("names");
        Console.ReadLine();
    }
    static IEnumerable<int> NumbersAndThrow()
    {
        yield return 1;
        yield return 2;
        yield return 3;
        throw new ApplicationException("报错啦");
        yield return 4;
    }
}
```

输出：

```
enumerable with exception - OnNext(1)
enumerable with exception - OnNext(2)
enumerable with exception - OnNext(3)
enumerable with exception - OnError:
    System.ApplicationException: Something Bad Happened
```

Rx 还提供了一个更简单的方法，直接订阅迭代类型：

```c#
IEnumerable<string> names = new[] { "AHpx", "Yoroion", "Weeknic", "GodLeaveMe" };
names.Subscribe(new ConsoleObserver<string>("subscribe"));
```

那么如何合并可观察数据流与迭代类型呢？

```c#
ChatClient client = new ChatClient();
IObservable<string> liveMessages = client.ObserveMessages("user","pass"); 
IEnumerable<string> loadedMessages = LoadMessagesFromDB(); // 假设从数据库加载数据

loadedMessages.ToObservable()
    .Concat(liveMessages)
    .SubscribeConsole("merged");
```

注意看，以上代码使用了 `Concat` 合并了两个流。在所有 Database 数据（Loaded Messages）被加载后，实时数据（Live Messages）才会被发给观察者。

```
merged - OnNext(loaded1)
merged - OnNext(loaded2)
merged - OnNext(live message1)
merged - OnNext(live message2)
```

你也可以用以下写法免于手动转换：

```c#
liveMessages
    .StartWith(loadedMessages)
    .SubscribeConsole("loaded first");
```

### 可观察流 → 迭代类型

为啥要把可观察流转化成迭代类型呢？因为有的方法只接受迭代类型。

```c#
var observable =
    Observable.Create<string>(o =>
    {
        o.OnNext("Observable");
        o.OnNext("To");
        o.OnNext("Enumerable");
        o.OnCompleted(); // 当所有OnNext中的值被消费后，线程会进入等待状态
        return Disposable.Empty;
    });
var enumerable = observable.ToEnumerable();
// 可观察流完成时，循环便会结束
foreach (var item in enumerable) 
{ 
    Console.WriteLine(item); 
}
```

把可观察流转换成 List：

```c#
var observable =
    Observable.Create<string>(o =>
    {
        o.OnNext("Observable");
        o.OnNext("To");
        o.OnNext("List");
        o.OnCompleted(); // 只有当Complete之后，List才会被发送给观察者
        return Disposable.Empty;
    });

IObservable<IList<string>> listObservable =
    observable.ToList();

listObservable
    .Select(lst => string.Join(",", lst)) // 使用逗号分隔
    .SubscribeConsole("list ready");
```

输出：

```
list ready - OnNext(Observable,To,List)
list ready - OnCompleted()
```

### 可观察流 → 字典

Rx 提供了如下方法：

```c#
IObservable<IDictionary<TKey, TSource>> ToDictionary<TSource, TKey>(this IObservable<TSource> source, Func<TSource, TKey> keySelector)
```

简单使用：

```c#
IEnumerable<string> cities = new[] { "London", "Tel-Aviv", "Tokyo", "Rome" };

var dictionaryObservable = cities
    .ToObservable()
    .ToDictionary(c => c.Length);

dictionaryObservable
    .Select(d => string.Join(",", d)) 
    .SubscribeConsole("dictionary");
```

输出：

```
dictionary - OnNext([6, London],[8, Tel-Aviv],[5, Tokyo],[4, Rome])
dictionary - OnCompleted()
```

但是当两个 Key 的值相同时，它就会报错。

如果你需要一 Key 对应多值，你就需要 **Lookup** 类型：

```c#
IObservable<ILookup<TKey, TSource>> ToLookup<TSource, TKey>(this IObservable<TSource> source, Func<TSource, TKey> keySelector)
```

长得和 `ToDictionary` 非常相似。

```c#
IEnumerable<string> cities = new[] { "London", "Tel-Aviv", "Tokyo", "Rome", "Madrid" }; 

var lookupObservable = cities
    .ToObservable()
    .ToLookup(c => c.Length); 

lookupObservable
    .Select(lookup =>
    {
        var groups = new StringBuilder(); 
        foreach (var grp in lookup) 
            groups.AppendFormat("[Key:{0} => {1}]",grp.Key,grp.Count()); 
        return groups.ToString(); 
    })
    .SubscribeConsole("lookup");
```

输出：

```
lookup - OnNext([Key:6 => 2][Key:8 => 1][Key:5 => 1][Key:4 => 1])
lookup - OnCompleted()
```

因为伦敦（London）和马德里（Madrid）的长度都是 6，Lookup Key 6 就有两个值。

## Rx 创建型操作符

### 可观察循环

使用 `Observable.Generate` 来创建类似迭代的数据流：

```c#
IObservable<TResult> Generate<TState, TResult>(
    TState initialState, 
    Func<TState, bool> condition, 
    Func<TState, TState> iterate, 
    Func<TState, TResult> resultSelector)
```

简单使用：

```c#
IObservable<int> observable = 
    Observable.Generate(
    0, //初始值
    i => i < 10, //条件
    i => i + 1, //迭代
    i => i*2); //数据处理
observable.SubscribeConsole();
```

你会得到：0, 2, 4, 6, 8, 10, 12, 14, 16, 18。

使用 `Observable.Range` 简化以上操作：

```c#
IObservable<int> Range(int start, int count)
```

顾名思义：

```c#
IObservable<int> observable = 
    Observable
    .Range(0, 10) 
    .Select(i => i*2);
```

### 读取文件

我们利用 `Generate` 方法来读取文件：

```c#
IObservable<string> lines = 
    Observable.Generate(
    File.OpenText("TextFile.txt"), //这个地方有瑕疵
    s => !s.EndOfStream, //迭代到文件末端
    s => s, //状态就是文件流本身
    s => s.ReadLine()); 

lines.SubscribeConsole("lines");
```

输出：

```
lines - OnNext(The 1st line)
lines - OnNext(The 2nd line)
lines - OnNext(The 3rd line)
lines - OnNext(The 4th line)
lines - OnCompleted()
```

但是捏，我们肯定知道 IO 流的资源没有及时释放。文件还保持在打开的状态。

在平时写 C# 中，我们一般会使用 `using var ...` 来自动释放资源。

在 Rx.NET 中，提供了 `Observable.Using`：

```c#
IObservable<string> lines = Observable.Using(
    () => File.OpenText("TextFile.txt"), 
    stream =>
    Observable.Generate(
        stream, //初始状态就是它自己
        s => !s.EndOfStream, 
        s => s, 
        s => s.ReadLine()) 
);

lines.SubscribeConsole("lines");
```

### 原始可观察流

有些 Rx 创建型操作符没啥用，但是可以拿来玩玩或者测试。

#### Return

```c#
Observable.Return("Hello World")
    .SubscribeConsole("Return");
```

输出：

```
Return - OnNext(Hello World)
Return - OnCompleted()
```

#### Never

一个永不结束的数据流：

```c#
Observable.Never<string>()
    .SubscribeConsole("Never");
```

#### Throw

一个运行即报错的流：

```c#
Observable.Throw<ApplicationException>(
    new ApplicationException("something bad happened"))
    .SubscribeConsole("Throw");
```

输出：

```
Throw - OnError:
    System.ApplicationException: something bad happened
```

#### Empty

```c#
Observable.Empty<string>()
    .SubscribeConsole("Empty");
```

输出：

```
Empty - OnCompleted()
```
