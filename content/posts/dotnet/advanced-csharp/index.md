---
title: "看看 C# 比较高级的特性"
description: 并发和异步
date: 2022-08-15T12:00:00
draft: false
slug: advanced-csharp
categories:
    - Dev
tags:
    - CSharp
---

## 前言

就是太久没写异步，有的用法忘了。所以写篇博客回顾一下。

## 线程 Thread

虽然在 C# 中 Task 的使用频率远超直接使用线程，但是有关线程的概念还是要复习一下的。

~~才不是因为我全忘了~~

### 创建线程 Create a Thread

```c#
using System;
using System.Threading;

var t = new Thread(WriteY);
t.Start();

for (var i = 0; i < 1000; i++)
{
    Console.Write("x");
}
void WriteY()
{
    for (var i = 0; i < 1000; i++) Console.Write("y");
}


//输出：
xxxxxxxxxxxxxyyyyyyyyyyyyyyyyyyyyy...
xxxxxxxxxxxxxxxxxxxxxxxxxyyyyyyyyy...
xxxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyyy...
yyyyyyyyyxxxxxxxxxxxxxxxxxxxxxxxxx...
```

我们可以通过 `Name` 属性来访问线程的名字：

```c#
WriteLine(Thread.CurrentThread.Name);
```

### Join 与 Sleep

你可以使用 `Join` 方法来等待线程结束：

```c#
Thread t = new Thread(Go);
t.Start();
t.Join();
Console.WriteLine("线程t结束");

void Go() { for (var i = 0; i < 1000; i++) Console.Write("y"); }
```

这段代码会打印 `y` 1000 次然后才会输出 `线程t结束`。

下面是 `Sleep` 的用法：

```c#
Thread.Sleep(TimeSpan.FromHours(1)); //睡一小时
Thread.Sleep(500)                    //睡500毫秒
```

特殊的是：

- `Thread.Sleep(0)` 可以放弃当前线程，主动将 CPU 移交给其它线程。

- `Thread.Yield()` 方法作用类似，只是移交**相同的**处理器。

### 阻塞线程 Blocking

查询阻塞状态：

```c#
bool isBlocked = (someThread.ThreadState & ThreadState.WaitSleepJoin) != 0;
```

> `ThreadState` 是一个 flags 枚举。

#### I/O 密集型 & 计算密集型 Bound

两者是 C# 并发中非常重要的概念。

- 等待事件发生的操作叫作 **I/O 密集型**（I/O-bound）
- 占用 CPU 来处理数据叫**计算密集型**（Compute-bound）

### 局部变量 & 共享状态 Local & Shared State

CLR 会分配给每个线程不同的内存栈，所以某一线程的局部变量是与其它线程隔绝的：

```c#
new Thread (Go).Start(); // 在新线程调用Go方法
Go();                    // 在主线程调用Go方法
void Go()
{
    //局部变量cycles
    for (int cycles = 0; cycles < 5; cycles++) Console.Write ('?');
}
```

结果可想而知，会输出 10 个问号「?」。

但是线程可以共享数据：如果它们有对同一对象或变量的引用：

```c#
bool _done = false;

new Thread (Go).Start();
Go()
;
void Go()
{
    if (!_done) { _done = true; Console.WriteLine ("Done"); }
}
```

两个线程共享了 `_done` 所以「Done」仅仅输出一次。

换成 Lambda 表达式也是可以共享的：

```c#
bool done = false;
ThreadStart action = () =>
{
    if (!done) { done = true; Console.WriteLine ("Done"); }
};
new Thread (action).Start();
action();
```

这引出了一个关键概念：**线程安全**！

### 线程锁 & 线程安全 Lock & Thread Safety

来看看锁🔒：

```c#
class ThreadSafe
{
    static bool _done;
    static readonly object _locker = new object();
    static void Main()
    {
        new Thread (Go).Start();
        Go();
    }

    static void Go()
    {
        lock (_locker)
        {
            if (!_done) { Console.WriteLine ("Done"); _done = true; }
        }
    }
}
```

其实这也不必多说，就是阻止两个线程同时修改某一变量罢了。

### 向线程传递数据

有的时候，我们想在线程开始的时候传递数据，可以这么做：

```c#
Thread t = new Thread ( () => Print ("Hello from t!") );
t.Start();

void Print (string message) => Console.WriteLine (message);
```

用 Lambda 表达式也行：

```c#
new Thread (() =>
{
    Console.WriteLine ("看！我跑在另一个线程里！");
    Console.WriteLine ("有手就行！");
}).Start();
```

#### 捕获变量 Captured Variables

一定要小心捕获变量：

```C#
for (int i = 0; i < 10; i++)
    new Thread (() => Console.Write (i)).Start();
```

输出结果是不确定的！结果：

```
0223557799
```

问题所在是变量 `i` 指向了循环中同一内存位置。所以变量 `i` 的值其实一直在被不同线程修改。

解决方案是使用临时变量：

```c#
for (int i = 0; i < 10; i++)
{
    int temp = i;
    new Thread (() => Console.Write (temp)).Start();
}
```

再来看看类似的问题：

```c#
string text = "t1";
Thread t1 = new Thread ( () => Console.WriteLine (text) );

text = "t2";
Thread t2 = new Thread ( () => Console.WriteLine (text) 
);
t1.Start(); t2.Start();
```

因为两个 Lambda 表达式捕获了同一变量 `text`，所以值 `t2` 会被输出两次。

### 异常处理

先看看**不正确**的写法：

```c#
try
{
    new Thread (Go).Start();
}
catch (Exception ex)
{
    // 永远不会到达这里！
    Console.WriteLine ("Exception!");
}

void Go() { throw null; } // 本应该抛出Null引用错误
```

每个线程有独立的执行路径，所以我们在线程之外捕获异常是没用的。

解决方案是把异常处理语句放入方法内：

```c#
new Thread (Go).Start();

void Go()
{
    try
    {
        throw null; 
    }
    catch (Exception ex)
    {
        // 这样就可以捕获了
    }
}
```

#### 集中异常处理 Centralized Exception Handler

在 WPF、UWP 或者 WinForm 应用里，我们可以订阅**全局**异常处理事件 `Application.DispatcherUnhandledException` 以及 `Application.ThreadException`。

### 前台线程 & 后台线程 Foreground & Background Threads

默认情况下，你显式创建的线程都是前台线程。程序结束，你的显式线程也会结束。

后台线程则不然，依然会保持运行。

你可以用 `IsBackground` 属性来操作前后台状态：

```c#
static void Main (string[] args)
{
   Thread worker = new Thread ( () => Console.ReadLine() );
   if (args.Length > 0) worker.IsBackground = true;
   worker.Start();
}
```

### 线程级别 Thread Priority

一个线程的 `Priority` 属性决定了它可以运行多久（相较于其它线程而言）：

```c#
enum ThreadPriority { Lowest, BelowNormal, Normal, AboveNormal, Highest }
```

### 发信号 Signaling

有时，你需要线程一直等待直到收到其它线程发送的通知，这就是 **Signaling**。

最简单的 Signaling 结构是 `ManualResetEvent` 。

在一个 `ManualResetEvent` 块中调用 `WaitOne` 方法，阻塞当前线程直到另一线程调用 `Set` 方法：

```c#
var signal = new ManualResetEvent (false);

new Thread (() =>
{
    Console.WriteLine ("Waiting for signal...");
    signal.WaitOne();
    signal.Dispose();
    Console.WriteLine ("Got signal!");
}).Start();


Thread.Sleep(2000);
signal.Set(); // 发送信号
```

### 线程在客户端应用 Threading in Rich Client Applications

在 WPF 或者 UWP 等等客户端的开发中，但我们要执行耗时的任务时，通常的做法是启动一个「Worker」线程。

当你想要从「Worker」线程更新 UI 时，你必须传递至 UI 线程。编程术语「编列 Marshal」专门指代这一行为。

- WPF：调用 `Dispatcher` 对象的 `BeginInvoke` 或 `Invoke` 方法

- UWP：调用 `Dispatcher` 对象的 `RunAsync` 或 `Invoke` 方法

- WinForm：调用控件的 `BeginInvoke` 或 `Invoke` 方法

这些方法的实质都是把你想要执行的方法推送到 UI 线程的消息队列中。

但是 `Invoke` 方法有一点特殊：它会**阻塞**线程直到消息被 UI 线程处理。因此它可以用来返回值。

```c#
partial class MyWindow : Window
{
    public MyWindow()
    {
        InitializeComponent();
        new Thread (Work).Start();
    }

    void Work()
    {
        Thread.Sleep (5000); // 假装耗时任务
        UpdateMessage ("The answer");
    }

    void UpdateMessage (string message)
    {
        Action action = () => txtMessage.Text = message;
        Dispatcher.BeginInvoke (action);
    }
}
```

### 同步上下文 Synchronization Contexts

WPF、UWP 等等框架都实现了这个类（子类）。

这个类被放在 `System.ComponentModel` 中，用来帮助我们进行 「Marshal」操作。

```c#
partial class MyWindow : Window
{
    SynchronizationContext _uiSyncContext;
    public MyWindow()
    {
        InitializeComponent();

        // 获取当前UI线程的同步上下文
        _uiSyncContext = SynchronizationContext.Current;
        new Thread (Work).Start();
    }

    void Work()
    {
        Thread.Sleep (5000); // 假装耗时操作
        UpdateMessage ("The answer");
    }

    void UpdateMessage (string message)
    {
        // Marshal委托至UI线程
        _uiSyncContext.Post (_ => txtMessage.Text = message, null);
    }
}
```

调用 `Post` 方法等价于调用 `BeginInvoke` 方法。

### 线程池 The Thread Pool

线程池就是用来方便多线程管理的。

有几点值得注意：

- 你不能为池线程设置 Name 属性，这会让 Debug 更加困难

- 池线程通常是**后台线程**

- 阻塞池线程会导致性能降低

你可以改变池线程的级别。

你可以通过 `Thread.CurrentThread.IsThreadPoolThread.` 来指定是否让线程在池里运行。

#### 进入线程池 Enter the pool

最简单的方式是：

```c#
Task.Run (() => Console.WriteLine ("你先别急，Task后面会讲"));
```

在 .NET Framework 4.0 以前的上古时期 C# 没有 Task 协程，是这样进入线程池的：

```c#
ThreadPool.QueueUserWorkItem (notUsed => Console.WriteLine ("Hello"));
```

> 隐式使用线程池的库：
> 
> - ASP.NET Core / Web API 应用
> 
> - System.Timers.Timer 和 System.Threading.Timer
> 
> - BackgroundWorker 类（传统）

#### 线程池卫生 Hygiene in the thread pool

线程池还有一个作用是保证 CPU 不会「超额认购 Oversubscription」。

「Oversubscription」指的是活跃线程数量超过 CPU 核心数量的状态。

总之，CLR 会通过排序任务队列和减速任务启动来阻止「超额认购」。

---

## 任务 Task

### 创建任务 Create a Task

```c#
Task.Run (() => Console.WriteLine ("Foo"));
Console.ReadLine(); //用来阻塞一下
```

#### 等待 Wait

和线程的 `Join` 类似：

```c#
Task task = Task.Run (() =>
{
    Thread.Sleep (2000);
    Console.WriteLine ("Foo");
});

Console.WriteLine (task.IsCompleted); // False
task.Wait(); // 阻塞直到Task结束
```

#### 耗时任务 LongRunning Task

CLR 默认会让 Task 运行在池线程（适用于短时计算的线程）。

为了运行耗时长的 Task：

```c#
Task task = Task.Factory.StartNew (() => ..., TaskCreationOptions.LongRunning);
```

### 返回值 Return Values

最简单的返回值写法：

```c#
Task<int> task = Task.Run (() => { Console.WriteLine ("Foo"); return 3; });
```

你可以通过 Task 的 `Result` 属性来获取返回值。这步操作将会阻塞线程直至 Task 结束：

```c#
int result = task.Result;
Console.WriteLine(result);
```

接下来，我们创建一个使用 LINQ 的 Task，用以计算 300 0000 以内的素数：

```c#
Task<int> primeNumberTask = Task.Run (() => 
    Enumerable.Range (2, 3000000).Count (n =>
        Enumerable.Range (2, (int)Math.Sqrt(n)-1).All (i => n % i > 0)));

Console.WriteLine ("Task running...");
Console.WriteLine ("The answer is " + primeNumberTask.Result);
```

### 异常 Exception

```c#
//开启一个抛出NullReferenceException的Task
Task task = Task.Run (() => { throw null; });
try
{
    task.Wait();
}
catch (AggregateException aex)
{
    if (aex.InnerException is NullReferenceException)
    Console.WriteLine ("Null!");
else
    throw;
}
```

### 后续 Continuations

顾名思义，就是当 Task 结束之后该干啥：

```c#
Task<int> primeNumberTask = Task.Run (() =>
    Enumerable.Range (2, 3000000).Count (n =>
        Enumerable.Range (2, (int)Math.Sqrt(n)-1).All (i => n % i > 0)));

var awaiter = primeNumberTask.GetAwaiter();
awaiter.OnCompleted (() =>
{
    int result = awaiter.GetResult();
    Console.WriteLine (result); 
});
```

---

## C# 异步编程

终于到了本章的重头戏了！

### Awaiting

`await` 关键字自动附加后续操作。

简单使用：

```c#
var result = await expression;
statement(s);
```

编译器会把这小段代码自动转化为：

```c#
var awaiter = expression.GetAwaiter();
awaiter.OnCompleted (() =>
{
    var result = awaiter.GetResult();
    statement(s);
});
```

来看看之前的代码：

```c#
Task<int> GetPrimesCountAsync (int start, int count)
{
    return Task.Run (() =>
        ParallelEnumerable.Range (start, count).Count (n =>
            Enumerable.Range (2, (int)Math.Sqrt(n)-1).All (i => n % i > 0)));
}
```

使用 `await` 关键字，我们可以如下调用：

```c#
int result = await GetPrimesCountAsync (2, 1000000);
Console.WriteLine (result);
```

但在编译前，我们需要给外层代码加一个 `async` 关键字：

```c#
async void DisplayPrimesCount()
{
    int result = await GetPrimesCountAsync (2, 1000000);
    Console.WriteLine (result);
}
```

`async` 关键字只能加给返回值类型是 `void`，`Task` 以及 `Task<TResult>` 的方法。

当返回值是空时：

```c#
await Task.Delay (5000);
Console.WriteLine ("Five seconds passed!");
```

#### 捕获局部变量 Capturing local state

`await` 的真正力量在于它可以出现在代码的任意位置（但不能在线程锁以及 `unsafe` 块中）：

```c#
async void DisplayPrimeCounts()
{
    for (int i = 0; i < 10; i++)
    Console.WriteLine (await GetPrimesCountAsync (i*1000000+2, 1000000));
}
```

在第一次执行 `GetPrimesCountAsync` 之后，变量 `i` 的值会被保存。

#### 在 UI 中等待 Awaiting in a UI

我们先从一个同步的代码开始：

```c#
class TestUI : Window
{
    Button _button = new Button { Content = "Go" };
    TextBlock _results = new TextBlock();
    
    public TestUI()
    {
        var panel = new StackPanel();
        panel.Children.Add (_button);
        panel.Children.Add (_results);
        Content = panel;
        _button.Click += (sender, args) => Go();
    }

    void Go()
    {
    for (int i = 1; i < 5; i++)
        _results.Text += GetPrimesCount (i * 1000000, 1000000) +
        " primes between " + (i*1000000) + " and " + ((i+1)*1000000-1) +
        Environment.NewLine;
    }
    
    int GetPrimesCount (int start, int count)
    {
        return ParallelEnumerable.Range (start, count).Count (n => Enumerable.Range (2, (int) Math.Sqrt(n)-1).All (i => n % i > 0));
    }
}
```
可以看到点击按钮就会阻塞线程。

接下来我们使用异步改写。

第一步是把计算素数的方法改成异步的：

```c#
Task<int> GetPrimesCountAsync (int start, int count)
{
    return Task.Run (() => ParallelEnumerable.Range (start, count).Count(n => Enumerable.Range(2, (int) Math.Sqrt(n)-1).All (i => n % i > 0)));
}
```
第二步是修改 `Go` 方法：

```c#
async void Go()
{
    _button.IsEnabled = false;
    for (int i = 1; i < 5; i++)
    _results.Text += await GetPrimesCountAsync (i * 1000000, 1000000) + " primes between " + (i*1000000) + " and " + ((i+1)*1000000-1) +
    Environment.NewLine;
    _button.IsEnabled = true;
}
```
再来另一个例子，这回我们想要从网络上异步地下载数据了。

重写 `Go` 方法：

```c#
async void Go() 
{
    _button.IsEnabled = false;
    string[] urls = "yoroion.github.io www.oreilly.com sinoahpx.github.io".Split();
    int totalLength = 0;
    try
    {
        foreach (string url in urls)
        {
            var uri = new Uri("http://" + url)
            byte[] data = await new WebClient().DownloadDataTaskAsync (uri);
            _results.Text += "Length of " + url + " is " + data.Length + Environment.NewLine;
            totalLength += data.Length;
        }
        _results.Text += "Total length: " + totalLength;
    }
    catch
    {
        _results.Text += "Error: " + ex.Message;
    }
    finally { _button.IsEnabled = true; }
}
```
我们附加在 UI 控件上的 Event Handler 在消息队列（message loop）中进行。

当我们的 `Go` 方法运行时，直至遇到 await 表达式，会跳转回消息队列来来响应其它事件。

### 编写异步方法 Writing Asynchronous Functions

编写异步方法，我们可以把空返回值的方法改成返回 Task：

```c#
async Task PrintAnswerToLife() //Task 替代了 void
{
    await Task.Delay (5000);
    int answer = 21 * 2;
    Console.WriteLine (answer);
}
```
可以发现我们并没有显式地返回 Task，编译器帮我们简化了：

```c#
async Task Go()
{
    await PrintAnswerToLife();
    Console.WriteLine("Done");
}
```
#### 返回 Task<TResult>

```c#
async Task<int> GetAnswerToLife()
{
    await Task.Delay (5000);
    int answer = 21 * 2;
    return answer; //直接返回整型
}
```
整体看一下代码:
```c#
async Task Go()
{
    await PrintAnswerToLife();
    Console.WriteLine ("Done");
}
async Task PrintAnswerToLife()
{
    int answer = await GetAnswerToLife();
    Console.WriteLine (answer);
}
async Task<int> GetAnswerToLife()
{
    await Task.Delay (5000);
    int answer = 21 * 2;
    return answer;
}
```
> 这也揭示了 C# 中编写异步方法的基本原则:
> 1. 先同步地编写代码
> 2. 将同步方法改成异步的
> 3. 返回值改成 Task 或 Task<TResult>

#### 并行 Parallelism

