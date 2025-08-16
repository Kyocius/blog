---
title: "写给 C# 开发者的 F# 光速入门指南"
date: 2022-08-26
draft: false
slug: fsharp-for-csharp-devs
description: Guide for C# devs to learn F# real FAST
categories:
    - Dev
tags:
    - FSharp

---

> 本文翻译自托管在 GitHub 上的 [2fsharp](https://github.com/knocte/2fsharp) 开源项目
> 
> 译版仅发表在知乎和 [本人博客](https://kyocius.github.io) 上，未经授权**禁止转载**

这个教程是基于实例来讲解的，将会花费你 15-30 分钟的时间，你将会学到 F# 这门语言 80% 的特性！

# 例 1：基本函数声明与实现

```csharp
public int GiveMeTheLength(string input)
{
    // 单行注释
    var result = input.Length;
    /* 多行注释 */
    return result;
}
```
变成了
```fsharp
let GiveMeTheLength(input) =
    // 单行注释
    let result = input.Length
    (* 多行注释 *)
    result
```

* 所有东西都是默认为 `public` 的，除非你显式声明 `private` 修饰符。
* 关键字 `var` 变成了 `let`（不仅仅用来声明变量，还用来声明函数）。
* 你不再需要 `return` 关键字。函数的最后一行就是返回值。
* 不再需要花括号，使用 4 格（或者 2 格）缩进，就像 Python 一样。
* 不再需要分号来标志一行结束。
* 类型声明是可选的，除了极少数情况编译器无法推断类型。
* 单行注释和 C# 相同，但是多行注释使用括号而不是斜杠。


如果你想要声明类型，需要这样写：

```fsharp
let GiveMeTheLength(input: string): int =
    let result: int = input.Length
    result
```


# 例 2：基本关键字和操作符

```csharp
using System;

class MainClass
{
    void Main()
    {
        int exitCode = 0;
        if (incomingChar == Environment.NewLine)
            exitCode = 1;
        else if (!(incomingChar == String.Empty))
            exitCode = 2;
        else if (incomingChar != "\t" && incomingChar.Length > 1)
            exitCode = 3;
        else
            exitCode = 4;
        Environment.Exit(exitCode);
    }
}
```
变成了
```fsharp
open System

let mutable exitCode: int = 0
if incomingChar = Environment.NewLine then
    exitCode <- 1
elif not (incomingChar = String.Empty) then
    exitCode <- 2
elif (incomingChar <> "\t" && incomingChar.Length > 1) then
    exitCode <- 3
else
    exitCode <- 4
Environment.Exit(exitCode)
```
* `using` 关键字变成了 `open` 关键字。
* `if (x) foo(); else if (y) bar(); else baz();` 模式变成了 `if x then foo() elif y then bar() else baz()`，条件语句也不再需要圆括号。但是使用了新的关键字 `then` 和 `elif`。
* 初始赋值（只读常量）操作符是 `=`。如果你需要为这个元素重新赋值，你需要将它标记为 `mutable` 然后使用 `<-` 操作符。
* 操作符 `=` 变成了比较符（不再需要像 C# 那样使用双等号 `==` 了）。
* 操作符 `!=` 变成了 `<>`。
* 操作符 `!` 变成了 `not`。
* 操作符 `&&` 和 `||` 在 F# 不变。
* 不再需要提供 Main() 方法这样的入口函数, 直接在一个 `.fsx` 文件中编写逻辑或者使用最新的 `.fs` 文件中编写语句以满足 F# 编译器要求。

通常，上面的代码其实不需要使用 `mutable`，就像如下写法：

```fsharp
open System

let exitCode =
    if incomingChar = Environment.NewLine then
        1
    elif not (incomingChar = String.Empty) then
        2
    elif (incomingChar <> "\t" && incomingChar.Length > 1) then
        3
    else
        4

Environment.Exit(exitCode)
```

（这和 C# 中的三元操作符类似，但是可读性更高）


# 例 3：基础集合

```csharp
var intArray = new int[] { 1, 2, 3 };
var intList = new List<int>({ 4, 5, 6 });
IEnumerable<int> sequenceOfIntegers = intList;
var dictionary = new Dictionary<int,string>() {
    { "One", 1 },
    { "Two", 2 }
}
```
变成了如下代码（其实你不需要声明类型，这里只是为了方便教学）：
```fsharp
let intArray: array<int> = [| 1; 2; 3 |]
let intList: List<int> = [ 4 ; 5 ; 6 ]
let sequenceOfIntegers: seq<int> = intList
let dictionary: IDictionary<string,int> = dict [ ("One", 1); ("Two", 2) ]
```
* 声明 Array/List/Dictionary 时，逗号变成了分号。
* `IEnumerable<T>` 变成了 `seq<'T>`（Sequence 的缩写）。
* 你使用一个 `dict` 来初始化一个`IDictionary<K,V>` 集合，但是在 F# 中你将会更倾向于使用 `Map<'K,'V>` 因为它是可变的。

你可能也注意到了，泛型类型需要一个前缀单引号「'」


# 例 4：代码块

```csharp
try {
    TrySomething(someParam);
} catch (SomeException ex) {
    if (SomeCondition(ex)) {
        DoSomethingElse(ex);
        throw new OtherException();
    } else
        throw;
} finally {
    MakeSureToCleanup(someParam);
}
```
变成了
```fsharp
try
    try
        TrySomething(someParam)
    with
    | :? SomeException as ex ->
        if SomeCondition(ex) then
            DoSomethingElse(ex)
            raise OtherException
        else
            reraise()

finally
    MakeSureToCleanup(someParam)
```
* `catch` 变成了 `with` 关键字。
* `throw X;` 语句变成了 `raise X`，然后 `throw;` 变成了调用 `reraise()` 函数。
* 但是，没有 `try-with-catch` 块！F# 只有 `try-with` 和 `try-finally` 块。所以我们只能用两个 `try` 嵌套。

你可能会想，这是 F# 的缺陷，但其实完整的 `try-catch-finally` 块很少用，尤其是有了 `using` 关键字：

```csharp
using (var reader = new StreamReader(someFile))
{
    DoStuff(reader);
}
```
这变成了
```fsharp
use reader = new StreamReader(someFile)
DoStuff(reader)
```
* 使用 `use` 就不再需要嵌套代码块了。
* 资源会自动释放（和 C# 的 using 相同）。


# 例 5：避免 Null 和忽略

```csharp
void Check(SomeType someParam1, SomeType someParam2)
{
    if (someParam1 != null)
        stringBuilder.Append(someParam1.ToString());

    if (someParam2 != null)
        stringBuilder.Append(String.Empty);
}
```
变成了
```fsharp
let Check(someParam1: Option<SomeType>, someParam2: Option<SomeType>): unit =

    match someParam1 with
    | Some(someValue) -> // like 'as' in C#, you cast and want the value
        let str = someValue.ToString()
        ignore(stringBuilder.Append(str))
    | None -> ()

    match someParam2 with
    | Some(_) -> // like 'is' in C#, you don't care about the value
        stringBuilder.Append(String.Empty) |> ignore
    | _ -> ()

```
在 C# 中你经常会到处写判空检查（没有编译时安全）。在 F# 中，你将使用更安全的 `Option<T>` 类型（与`Nullable<T>` 类似但是更棒）和 `match` 表达式（模式匹配）。

* 当你不想返回任何东西时，在 C# 你会写 `void` 表示没有声明类型。但是在 F# 中你需要返回一个特殊的类 `unit`，它的默认值为 `()`。
* 一个 `match-with` 代码块基本上和 switch 代码块类似，但是更加简洁明了，因为它包含了 Casting（SomeValue）
* 这里有三个方式来实现忽略：
  * 比方说，我们不关心 `Append()` 函数的返回值，在 C# 中我们就直接忽略，但是在 F# 中我们需要显式地忽略它，使用 `ignore()` 这个魔法方法。
  * Match 表达式的下划线：它就像 C# 的 `switch` 中的 `default` 分支。
  * 在 `Some(_)` 中的下划线表示，我们想保证一个值不是空，但我们又不关心它的具体值（就像 C# 中的 `is` 操作符）。
* 管道操作符：`ignore(x)` 与 `x |> ignore` 等价。


# 例 6：类型

下面这个类使用 F# 就简单很多。

（译者注：其实可以使用 C# 6.0 的自动完成属性或者 C# 9.0 的记录类）

```csharp
public class Foo
{
    public Foo (int bar, string baz)
    {
        this.bar = bar;
        this.baz = baz;
    }

    readonly int bar;
    public int Bar
    {
        get { return bar; }
    }

    readonly string baz;
    public string Baz
    {
        get { return baz; }
    }
}

static class FooFactory
{
    static internal Foo CreateFoo()
    {
        return new Foo(42, "forty-two");
    }
}
```

在 F# 中就一行：

```fsharp
type Foo = { Bar: int; Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42; Baz = "forty-two" }
```

由此可见:
* 没有动作行为的类（就像上面的 Foo 类）叫作「记录类 Record」，长得像 C# 的结构体但是它们依然是引用类型，并且被保存在堆上。记录类是不可变的。也就是说，一旦你创建它们，就不能再修改它们的值了。
* 静态类变成了「modules」，如上 `FooFactory` 类。
* 在 F# 中，不需要使用 `new` 关键字，除非这个类实现了 `IDisposable` 接口。


# 例 7：函数的顺序很重要，循环依赖是万恶之源

作为 C# 开发者，你当然知道下面的代码可以通过编译：

```csharp
static class Foo
{
    static void Bar()
    {
        if (Baz())
            Bar();
    }

    static bool Baz()
    {
        return false;
    }
}
```

理所应当！下面的代码也可以通过编译：

```csharp
static class Foo1
{
    public static void Bar()
    {
    }

    static void Baz()
    {
        Foo2.Baz();
    }
}

static class Foo2
{
    static void Bar()
    {
        Foo1.Bar();
    }

    public static void Baz()
    {
    }
}
```
也许你明白我说到哪里了。

后一段 C# 代码可以通过编译，是因为 C# 允许循环依赖（Circular Dependencies）。

但是这句话半对半错，因为在 C# 中循环依赖是合法的，但是在 .NET 程序集里不合法。如果程序集 A 依赖于程序集 B，你不能从 A 引用 B。

这个模块性原则将会阻止你写出上面的代码，因为在未来你将不能将它们分为两个程序集（你不能将 Foo1 放进一个程序集里，然后把 Foo2 放进另一个程序集，因为在 .NET 中这并不合法）。

之后，你需要了解的是，在 F# 中，即使在同一程序集里，循环依赖**也不合法**！

通过强制你在类型 B 之前声明 类型 A，防止你在之后的函数中调用前面的函数时出错。

因此，下面的 F# 编译时会报错：

```fsharp
module Foo1 =
    let Bar() =
        ()

    let Baz() =
        Foo2.Baz()

module Foo2 =
    let Bar() =
        Foo1.Bar()

    let Baz() =
        ()
```

为什么呢？来看看它的报错：

* Error FS0039: The value, namespace, type or module 'Foo2' is not defined. (Referring to Foo1.Baz implementation.)

这个报错是不能修复的，除非我们直接停止使用循环依赖。

（还有一些特殊情况，比如使用 `via` 和 `rec` 关键字，但是在本教程中并不会讲解，因为太高级了）

F# 编译器严格限制了我们不能循环依赖，这意味着如果函数 A 调用了函数 B，那么函数 B 就必须在 A 之前声明

（如果这两个函数在同一个文件中，那么函数 B 就是在函数 A 上面；如果这两个函数在同一 F# 工程下不同文件中，那么 `B.fs` 必须比 `A.fs` 早列出）

因此，下面的代码也不会通过编译：

```fsharp
module Foo =
    let Bar() =
        if Baz() then
            Bar()

    let Baz(): bool =
        false
```

它抛出了如下错误：

* Error FS0039: The value or constructor 'Baz' is not defined. (Referring to Foo.Bar implementation.)
* Error FS0039: The value or constructor 'Bar' is not defined. (Referring to Foo.Bar implementation.)

正如我们刚刚学到的，这更容易解决；只要先声明 `Baz` 即可。

而为了让一个函数能够调用自己（在某种程度上，它也是一个循环依赖），我们使用 `rec` 关键字（意思就是「递归」)：

```fsharp
module Foo =
    let Baz(): bool =
        false

    let rec Bar() =
        if Baz() then
            Bar()
```


# 例 8：委托、匿名函数···我的老天爷！

在 C# 的早期版本中，需要通过委托类型和匿名函数传递函数。请看下面有 4 种组合的例子：

```csharp
static class SomeOldCsharpClass
{
    delegate void ProcedureWithNoReturnValueAndNoArguments();

    static void DelegateReception1(ProcedureWithNoReturnValueAndNoArguments dlg)
    {
        dlg.Invoke();
    }

    static void SendingAnonymousMethodAsDelegate1()
    {
        DelegateReception1(delegate ()
        {
            Console.WriteLine("hello 1");
        });
    }

    delegate void ProcedureWithNoReturnValueAndOneArg(string foo);

    static void DelegateReception2(ProcedureWithNoReturnValueAndOneArg dlg)
    {
        string bar = "baz";
        dlg.Invoke(bar);
    }

    static void SendingAnonymousMethodAsDelegate2()
    {
        DelegateReception2(delegate (string foo)
        {
            Console.WriteLine("hello 2 " + foo);
        });
    }

    delegate int FunctionWithOneReturnValueAndNoArguments();

    static void DelegateReception3(FunctionWithOneReturnValueAndNoArguments dlg)
    {
        int result = dlg.Invoke();
    }

    static void SendingAnonymousMethodAsDelegate3()
    {
        DelegateReception3(delegate ()
        {
            Console.WriteLine("hello 3");
            return 3;
        });
    }

    delegate long FunctionWithOneReturnValueAndOneArg(double foo);

    static void DelegateReception4(FunctionWithOneReturnValueAndOneArg dlg)
    {
        double bar = 4.0;
        long result = dlg.Invoke(bar);
    }

    static void SendingAnonymousMethodAsDelegate4()
    {
        DelegateReception4(delegate (double foo)
        {
            Console.WriteLine("hello 4 " + foo);
            return 4;
        });
    }
}
```

但后来新版本的 C# 出现了更优雅的方式（得益于局部函数）：

```csharp
static class SomeNewCsharpClass
{
    static void SendingAnonymousMethodAsDelegate1()
    {
        void DelegateReception1(Action dlg)
        {
            dlg.Invoke();
        }

        DelegateReception1(() =>
        {
            Console.WriteLine("hello 1");
        });
    }

    static void SendingAnonymousMethodAsDelegate2()
    {
        void DelegateReception2(Action<string> dlg)
        {
            string bar = "baz";
            dlg.Invoke(bar);
        }

        DelegateReception2((string foo) =>
        {
            Console.WriteLine("hello 2 " + foo);
        });
    }

    static void SendingAnonymousMethodAsDelegate3()
    {
        void DelegateReception3(Func<int> dlg)
        {
            int result = dlg.Invoke();
        }

        DelegateReception3(() =>
        {
            Console.WriteLine("hello 3");
            return 3;
        });
    }

    static void SendingAnonymousMethodAsDelegate4()
    {
        void DelegateReception4(Func<double, long> dlg)
        {
            double bar = 4.0;
            long result = dlg.Invoke(bar);
        }

        DelegateReception4((double foo) =>
        {
            Console.WriteLine("hello 4 " + foo);
            return 4;
        });
    }
}
```

最有趣的改进就是 .NET 预定义的 `Action` 和 `Func` 类型，以及 Lambda 表达式。

好消息是在 F# 中，函数是一等公民。因此代码更加简单：

```fsharp
module SomeFsharpModule =

    let SendingAnonymousMethodAsDelegate1() =
        let DelegateReception1(dlg: unit->unit) =
            dlg()

        DelegateReception1(fun _ ->
            Console.WriteLine("hello 1")
        )

    let SendingAnonymousMethodAsDelegate2() =
        let DelegateReception2(dlg: string->unit) =
            let bar = "baz"
            dlg(bar)

        DelegateReception2(fun bar ->
            Console.WriteLine("hello 2 " + bar)
        )

    let SendingAnonymousMethodAsDelegate3() =
        let DelegateReception3(dlg: unit->int) =
            let result = dlg()
            ()

        DelegateReception3(fun _ ->
            Console.WriteLine("hello 3")
            3
        )

    let SendingAnonymousMethodAsDelegate4() =
        let DelegateReception4(dlg: double->int64) =
            let bar = 4.0
            let result = dlg(bar)
            ()

        DelegateReception4(fun bar ->
            Console.WriteLine("hello 4 " + bar.ToString())
            int64(4)
        )
```

正如你所看到的，BCL 的 `System.Function` 和 `System.Action` 变成了原生 F# 语法。

通过 `->` 符号来表示参数和返回值，例如 `TArg1->TResult`。请记住，`void` 在 F# 中是 `unit`。

同样重要的是，C# 的 `(...) => { ... }` 变成了 `fun ... -> ...`


# 例 9：元组、局部应用以及柯里化

如果你没有做过函数式编程，你可能会对其中的一些概念感到害怕，比如「局部应用 Partial Application」「柯里化 Currification」。

但事实是，它们并不是那么复杂的概念。为了正确解释它们，我们需要先解释元组，以及为什么不建议在 F# 中滥用它们。

（事实上，你不能用元组来使用局部应用！稍后将详细介绍）

让我们先看看下面这段使用 `out` 参数的 C# 代码：

```csharp
int anInteger;
if (int.TryParse(someString, out anInteger)) {
    DoSomethingWithAnInteger(anInteger);
} else {
    DoSomethingElse();
}
```
如果你尝试将上面的 C# 代码转译成 F#，你首先会有一个疑问：我们怎样声明一个变量但不给它赋值呢？相信我，F# 是不可能做到的。

你可能会想到先随便给变量 `anInteger` 赋一个值，反正最后会被 `TryParse()` 方法覆写。但是这种方式不够优雅，尤其是把变量标记为 `mutable`，不符合 F# 的语言习惯。

最好的方法是使用「积极模式 Acitve Pattern」——将上面多个函数转换成一个函数，并且同时虚拟地返回两个值：

```fsharp
match int.TryParse(someString) with
| (true, anInteger) -> DoSomethingWithAnInteger(anInteger)
| (false, _) -> DoSomethingElse()
```

积极模式为 F# 编译器提供了语法糖，将 `bool TryParse(string,out int)` 转换成了类似 `(bool,int) TryParse(string)` 的函数，而 `(bool,int)` 是一个元组！

所以在 F# 中，创建任意长度的元组是很容易的事情，不需要使用 `System.Tuple<X,Y,...>`。

这让我想起了新版本 C# 对于元组的改进——你可以在函数的签名里使用元组而不需要用 `Tuple`。

来检查一下我说的是什么意思吧。这是 C# 的老式写法：

```csharp
bool ReceiveTuple(Tuple<string,int> aTuple)
{
    var counter = aTuple.Item2++;
    Console.WriteLine(aTuple.Item1);
    var newTuple = new Tuple<string,int>(aTuple.Item1, counter)
    ReceiveTuple(newTuple);
    return true;
}
```

如果我们想要一个变量指向这个函数，它的类型就应是 `Func<Tuple<string,int>,bool>`。

新式的 C# 写法（会被编译成 `ValueTuple<X,Y,...>`）：

```csharp
bool ReceiveTuple((string str, int i) aTuple)
{
    var counter = aTuple.i++;
    Console.WriteLine(aTuple.str);
    var newTuple = (aTuple.str, counter);
    ReceiveTuple(newTuple);
    return true;
}
```

现在我们有了一个指向这个函数的变量，它的类型是 `Func<ValueTuple<string,int>,bool>`。

使用 F#：

```fsharp
let rec ReceiveTuple(str: string, i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    let newTuple = (str, counter)
    ReceiveTuple (newTuple)
    true
```
在这种情况下，你引用这个函数的 F# 类型是 `string*int->bool`；所以我们学到了一个新的符号：星号「*」在其它语言里代表指针，但在 F# 中它只是元组中类型的分隔符。

你是否注意到元组是如何融入到 F# 中那些看似正常的参数中的？事实上，本指南到现在为止，我们在 F# 中写的所有接收多参的函数，实际上都在使用元组。不用元组能在 F# 中写出的上述方法吗？当然可以，只是省略了逗号：

```fsharp
let rec ReceiveNonTuple (str: string) (i: int) =
    let counter = i + 1
    Console.WriteLine(str)
    ReceiveNonTuple str counter
    true
```

函数 `ReceiveTuple` 和 `ReceiveNonTuple` 之间有什么区别？两者都接收相同数量的参数，并且具有相同的类型。但是，第一个函数的参数是 F# 元组，而第二个函数的参数则是以 F# 语言惯用方式声明的（以「柯里化」的形式）。

为什么后者在 F# 中更符合语言习惯？因为 `ReceiveTuple` 不能用于局部应用，而`ReceiveNonTuple` 可以，因为它使用的是「柯里化」风格。在这种情况下，函数的类型不是 `string*int->bool`，而是 `(string->int)->bool`。

那为什么 `(string->int)->bool` 比 `string*int->bool` 更好呢？

后者允许与 C# 的互操作（因为这是以 CIL 级别传递参数的方式），但前者允许以非常直接和简洁的方式进行局部应用（我们将在后面看到，部分应用在 C# 中也是可能的，但方式非常复杂，可读性/维护性很差）。

因此，不再多说，让我们看看一个非常简单的部分应用的例子。假设我们想创建一个乘法函数，接收两个整数并返回一个整数：

```fsharp
let Multiply (x: int) (y: int): int =
    x * y
```

现在，如果我们想写一个函数将一个数字的值增加一倍，而不需要重复乘法函数的任何实现细节，我们可以简单地这样写：

```fsharp
let Double (x: int): int =
    Multiply 2
```

当我们只向函数 `Multiply` 传递一个参数时会发生什么？

让我们看看它的原始签名：`(int->int)->int`。「柯里法则」告诉我们，在类型表达式中使用小括号实际上是不需要的（或者把它们放在其他地方会得到一个等价的表达式）。这意味着我们也可以这样写 `int->int->int` 或者这样写 `int->（int->int）`。因此，向一个原本接收两个参数的函数只传递一个参数，实际上会导致返回另一个函数。

我们可以这样更好地理解它：

```fsharp
let Double (x: int): int =
    let doubleFunc = Multiply 2
    let result = doubleFunc x
    result
```

或者这样（冗长的类型声明）：

```fsharp
let Double (x: int): int =
    let doubleFunc: int->int = Multiply 2
    let result: int = doubleFunc x
    result
```

这是一个非常简单的例子，也许能让你明白局部应用的强大之处。但它是函数式编程中的依赖注入等操作的基础。随着时间的推移，你可能只会掌握它所带来的灵活性，但至少我们已经可以向你展示它在 F# 中的简单性。在 C# 中，我们只能这样来实现：

```csharp
static Func<int, Func<int, int>> Multiply()
{
    return (x) => {
        (y) => {
            x * y;
        };
    };
}

static Function<int,int> Double()
{
    return Multiply().Invoke(2);
}

static int Main()
{
    var someInt = 3;
    return Double().Invoke(someInt);
}
```

你的大脑能转过弯儿来吗？对我来说，这比阅读 F# 代码难理解多了。

关于局部应用及其用处的最后说明：你使用 F# 的时间越长，你就会意识到局部应用实际上是一种简化的依赖注入/控制反转（DI / IoC）方式，详见[《局部应用就是依赖注入》](https://blog.ploeh.dk/2017/01/30/partial-application-is-dependency-injection/)一文。


# 例 10：字符串插值

在早期的 C# 版本中，你需要这样写：

```csharp
var aStringToShowToTheUser = String.Format("Hello {0}, I see you are {1} years old", name, age);
```

这有两个问题：在大型代码库中，许多变量和元素被包含在一个字符串中，我们索引了一个没有提供的变量（例如 `{2}`）很容易发生，或者我们提供的变量没有隐式转换为字符串。

在 F# 中是这样的：

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```

哪个更好？
* 如果你提供的参数少于（或多于）在字符串中插值所需的参数，你将得到一个编译器错误，而不是在运行时出现异常。
* 你需要通过 `%` 后面的字母来指定字符串中元素的类型，如果它与为同一位置提供的元素的类型不匹配，那么你会得到一个编译器错误（而不是一个无用的元素字符串表示）。
* 你需要更少的小括号（参见本指南中的最后一个例子）。

# 例 11：异步代码

一段简单的 C# 异步代码：

```csharp
    static class MainClass
    {
        public class Toast {
            public bool IsYummy()
            {
                return true;
            }
        }

        static async Task<Toast> ToastBreadAsync()
        {
            var task = Task.Run(() =>
                new Toast()
            );
            return await task;
        }

        static void ApplyButter(Toast toast) { /* TODO */ }

        static void ApplyJam(Toast toast) { /* TODO */ }

        static async Task<Toast> MakeToastWithButterAndJamAsync()
        {
            var toast = await ToastBreadAsync();
            ApplyButter(toast);
            ApplyJam(toast);
            return toast;
        }

        public static async Task Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            var toast = await MakeToastWithButterAndJamAsync();
            Console.WriteLine("Bye World!" + toast.IsYummy());
        }
    }
}
```

在 F# 中变成了：

```fsharp
type Toast() =
    member this.IsYummy() =
        true

let ToastBread(): Async<Toast> =
    async {
        return Toast()
    }

let ApplyButter(toast) =
    () //TODO

let ApplyJam(toast) =
    () //TODO

let MakeToastWithButterAndJam() =
    async {
        let! toast = ToastBread()
        ApplyButter(toast)
        ApplyJam(toast)
        return toast
    }


[<EntryPoint>]
let main(argv) =
    Console.WriteLine("Hello World!")
    let toast = MakeToastWithButterAndJam()
                |> Async.RunSynchronously
    Console.WriteLine ("Bye world!" + (toast.IsYummy().ToString()))
    0 // return an integer exit code
```

关键不同点：

* 在 C# 中，当你调用一个异步方法时，你会得到一个 `Task<T>` 对象，它代表正在进行的工作，并且已经被启动。但在 F# 中，Job 是由一个 `Async<'T>` 对象表示的，它还没有被启动（你可以在之后决定如何启动它；例如，在上面例子中，它用 `Async.RunSynchronously` 来启动）。
* 在 F# 的 `let` 语句后面加上惊叹号 `!` 等价于 C# 的 `await`。
* 在 C# 中，你将计算量大的同步方法包装在 `Task.Run()` 中，将其转换为异步方法。在 F# 中，你只需用 `async{}` 块（一个计算表达式）来包装它们。

如果我们稍微改变上面的 C# 代码，引入非泛型 Task 对象和并行化（假设我们有两个烤面包机）：

```csharp
public class Ingredients
{
}

public class Toast {
    public Toast(Ingredients i)
    {
    }
}

static async Task<Toast> ToastBreadAsync(Ingredients i)
{
    var task = Task.Run(() =>
        new Toast(i)
    );
    return await task;
}

static async Task<Toast[]> Make2ToastsAsync(Ingredients i)
{
    var toast1 = ToastBreadAsync(i);
    var toast2 = ToastBreadAsync(i);
    return await Task.WhenAll(toast1, toast2);
}

static async Task MakeToastsAsync()
{
    var i = await GatherIngredients();
    await Make2ToastsAsync(i);
}

public static async Task<Ingredients> GatherIngredients()
{
    return await Task.FromResult(new Ingredients());
}

public static async Task Main(string[] args)
{
    Console.WriteLine("Hello World!");
    await MakeToastsAsync();
    Console.WriteLine("Bye World!");
}
```

在 F# 中，这变成了：

```fsharp
type Ingredients () = class end
type Toast (i: Ingredients) = class end

let ToastBread(i): Async<Toast> =
    async {
        return Toast(i)
    }

let GatherIngredients () =
    async { return Ingredients() }

let Make2Toasts(i) =
    async {
        let twoJobs: List<Async<Toast>> = [ToastBread(i); ToastBread(i)]
        let! _ = Async.Parallel(twoJobs)
        return ()
    }

let MakeToasts() =
    async {
        let! i = GatherIngredients()
        do! Make2Toasts(i)
    }


[<EntryPoint>]
let main(argv) =
    Console.WriteLine("Hello World!")
    MakeToasts()
        |> Async.RunSynchronously
    Console.WriteLine("Bye world!")
    0 // return an integer exit code
```

正如你所看到的：
* 在 C# 中处理非泛型 Task，在 F# 中意味着使用 `unit` 作为 `Async` 的泛型参数：`Async<unit>`。不要使用 `let! x = ...`，你只需要 `do!`。
* `Task.WhenAll` 的等价物是 `Async.Parallel`。


# 例 12：编写更少符号，提升 F# 脚本的可阅读性！

相信你已经明白了 F# 中元组和柯里化参数的区别。后者更推荐使用。

你可能明白，写这么多小括号其实只是为了映射元组中的东西。事实上，这是刚从 C# 转到 F# 的思维惯性。

F# 的简洁之处：

* 没有那么多的括号，因为你不再使用元组了。
* 不需要使用分号，如果你只使用 EOL 分隔符。
* 不需要使用大括号（在 F# 中处理记录类型时才需要它们）。
* 让 F# 编译器推断类型，就不需要多次使用冒号 `:`。
* 更多地使用管道运算符 `|>`，以避免在一大串代码的右边还要写一堆小括号。
* 在 F# 中的 `if` 表达式中不需要小括号（与 C# 相反）。

有了这些要点，我们来重写之前的所有 F# 实例：

```fsharp
open System

let exitCode =
    if incomingChar = Environment.NewLine then
        1
    elif not (incomingChar = String.Empty) then
        2
    elif incomingChar <> "\t" && incomingChar.Length > 1 then
        3
    else
        4

Environment.Exit exitCode
```

```fsharp
let intArray = 
        [| 1
           2
           3 |]
let intList =
        [ 4
          5
          6 ]
let sequenceOfIntegers = intList
let dic = dict [ ("One", 1)
                 ("Two", 2) ]
```

```fsharp
try
    try
        TrySomething someParam
    with
    | :? SomeException as ex ->
        if SomeCondition ex then
            DoSomethingElse ex
            raise OtherException
        else
            reraise()

finally
    MakeSureToCleanup someParam
```

```fsharp
use reader = new StreamReader someFile
DoStuff reader
```

```fsharp
let Check someParam1 someParam2 =

    match someParam1 with
    | Some someValue -> // like 'as' in C#, you cast and want the value
        let str = someValue.ToString()
        stringBuilder.Append str |> ignore
    | None -> ()

    match someParam2 with
    | Some _ -> // like 'is' in C#, you don't care about the value
        stringBuilder.Append String.Empty |> ignore
    | _ -> ()

```

```fsharp
type Foo = 
    { Bar: int
      Baz: string }

module FooFactory =
    let internal CreateFoo () =
        { Bar = 42
          Baz = "forty-two" }
```

```fsharp
module Foo =
    let Baz() =
        false

    let rec Bar() =
        if Baz() then
            Bar()
```

```fsharp
module SomeFsharpModule =

    let SendingAnonymousMethodAsDelegate1() =
        let DelegateReception1 dlg =
            dlg()

        DelegateReception1(fun _ ->
            Console.WriteLine "hello 1"
        )

    let SendingAnonymousMethodAsDelegate2() =
        let DelegateReception2 dlg =
            let bar = "baz"
            dlg bar

        DelegateReception2(fun bar ->
            Console.WriteLine("hello 2 " + bar)
        )

    let SendingAnonymousMethodAsDelegate3() =
        let DelegateReception3 dlg =
            let result = dlg()
            ()

        DelegateReception3(fun _ ->
            Console.WriteLine "hello 3"
            3
        )

    let SendingAnonymousMethodAsDelegate4() =
        let DelegateReception4 dlg =
            let bar = 4.0
            let result = dlg bar
            ()

        DelegateReception4(fun bar ->
            Console.WriteLine("hello 4 " + bar.ToString())
            int64 4
        )
```

```fsharp
match int.TryParse someString with
| true, anInteger -> DoSomethingWithAnInteger anInteger
| false, _ -> DoSomethingElse()
```

```fsharp
let rec ReceiveNonTuple str i =
    let counter = i + 1
    Console.WriteLine str
    ReceiveNonTuple str counter
    true
```

```fsharp
let Multiply x y =
    x * y

let Double x =
    let doubleFunc = Multiply 2
    let result = doubleFunc x
    result
```

```fsharp
let aStringToShowToTheUser = sprintf "Hello %s, I see you are %i years old" name age
```

```fsharp
type Ingredients () = class end
type Toast (i: Ingredients) = class end

let ToastBread i: Async<Toast> =
    async {
        return Toast i
    }

let GatherIngredients () =
    async { return Ingredients() }

let Make2Toasts i =
    async {
        let twoJobs: List<Async<Toast>> = [ToastBread i; ToastBread i]
        let! _ = Async.Parallel twoJobs
        return ()
    }

let MakeToasts() =
    async {
        let! i = GatherIngredients()
        do! Make2Toasts i
    }


[<EntryPoint>]
let main argv =
    Console.WriteLine "Hello World!"
    MakeToasts()
        |> Async.RunSynchronously
    Console.WriteLine "Bye world!"
    0 // return an integer exit code
```


------------------------------------------------------

祝贺你! 你学习了足够的知识，大概可以理解 80% 的 F# 代码了。

或者说 80% 的简单 F# 代码，也就是大多数 F# 脚本中所使用的。

我可以向你解释如何在 F# 中建立相当于类（有行为、构造函数、属性）或结构（值类型和堆栈分配）但是：
1. 这些不是 F# 的惯用写法
2. 大多数脚本甚至不需要类型，它们只需要函数、值和调用（如果你把 F# 当脚本用）

无论如何，我建议下一步：
- 观看这个 [讲座](https://www.youtube.com/watch?v=HQ887aOZITY)
- 看看这篇 [文章](https://www.compositional-it.com/news-blog/5-features-that-c-has-that-f-doesnt-have/)

如果过了一段时间，你还在为从 C# 转换至 F# 而挣扎，也可以时不时地使用这个 [工具](https://github.com/willsam100/FShaper)。



