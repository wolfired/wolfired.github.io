---
layout: post
---

[TOC]

__[PART 1: GETTING STARTED][Source Link]__
---

# __Introduction and Concepts__

C#支持通过多线程并行执行代码, 一个线程就是一个独立的执行路径, 与其它线程同时运行.

一个C# _client_ 程序(Console程序, WPF程序或Windows Forms程序)开始于一个由CLR和OS自动创建的线程( _main_ 线程), 通过创建附加线程实现多线程运行. 下面是一个简单例子:

>本教程所有的例子都假设导入了以下命名空间

```csharp
using System;
using System.Threading;
```

```csharp
class ThreadTest
{
  static void Main()
  {
    Thread t = new Thread (WriteY);          // Kick off a new thread
    t.Start();                               // running WriteY()
 
    // Simultaneously, do something on the main thread.
    for (int i = 0; i < 1000; i++) Console.Write ("x");
  }
 
  static void WriteY()
  {
    for (int i = 0; i < 1000; i++) Console.Write ("y");
  }
```

```shell
#output
xxxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxyyyyyyyyyyyyy
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
yyyyyyyyyyyyyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
...
```

_main_ 线程创建了一个新的线程 __t__ , 它将重复打印字符"y". 同时, _main_ 线程重复打印字符"x":

![][NewThread.png]

线程一旦启动, 线程的 __IsAlive__ 属性返回 __true__, 直到线程运行结束. 传递给 __Thread__ 构造函数的 __delegate__ 执行完毕标志线程运行结束. 线程一旦结束,将不能再次启动.

CLR给各个线程分配私有memory stack用于隔离线程内部的本地变量. 在下个例子中, 我们定义带一个本地变量的方法, 在 _main_ 线程和新建线程中同时调用:

```csharp
static void Main() 
{
  new Thread (Go).Start();      // Call Go() on a new thread
  Go();                         // Call Go() on the main thread
}
 
static void Go()
{
  // Declare and use a local variable - 'cycles'
  for (int cycles = 0; cycles < 5; cycles++) Console.Write ('?');
}
```
```shell
#output
??????????
```

各个线程在其私有memory stack里创建变量cycles的副本, 所以可以预见上面的输出是10个问号.

如果线程有着指向相同对象的引用,那它们将共享这个对象数据. 如:

```csharp
class ThreadTest
{
  bool done;
 
  static void Main()
  {
    ThreadTest tt = new ThreadTest();   // Create a common instance
    new Thread (tt.Go).Start();
    tt.Go();
  }
 
  // Note that Go is now an instance method
  void Go() 
  {
     if (!done) { done = true; Console.WriteLine ("Done"); }
  }
}
```

因为两个线程调用同一个 __ThreadTest__ 实例的 __Go()__ 方法, 所以它们共享 __done__ 属性. 其结果就导致"Done"有可能只显示一次而不是一定显示再次:

```shell
#output
Done
```

静态属性是线程间共享数据的另一种方式. 下面的例子把 __done__ 换成静态属性:

```csharp
class ThreadTest 
{
  static bool done;    // Static fields are shared between all threads
 
  static void Main()
  {
    new Thread (Go).Start();
    Go();
  }
 
  static void Go()
  {
    if (!done) { done = true; Console.WriteLine ("Done"); }
  }
}
```

上面两个例子为我们指出了一个关键概念: 线程安全. 因为它们的输出实际上是不确定的: "Done"有可能打印两次(尽管概率比较低). 但,如果我们把调整 __Go__ 内的语句顺序, 这个概率将大幅提高:

```csharp
static void Go()
{
  if (!done) { Console.WriteLine ("Done"); done = true; }
}
```
```shell
#output
Done
Done   (usually!)
```

问题的关键在于当一个线程评估 __if__ 语句的同时另一个线程现在执行 __WriteLine__ 语句, 此时后一个线程还无法将 __done__ 的值置true.

解决的办法是在读写共享属性前获取一把[独占锁][Locking]. C#提供了用于此目的的[lock][Locking]语句:

```csharp
class ThreadSafe 
{
  static bool done;
  static readonly object locker = new object();
 
  static void Main()
  {
    new Thread (Go).Start();
    Go();
  }
 
  static void Go()
  {
    lock (locker)
    {
      if (!done) { Console.WriteLine ("Done"); done = true; }
    }
  }
}
```

当两个线程同时抢夺同一把锁(在这里是 __locker__ ), 在锁被释放前, 其中一个线程将会等待或[阻塞][Blocking]. 在这个例子里, 锁确保了在同一时刻只有一个线程可以进入临界代码, 而"Done"只会打印一次. 在多线程的不确定状态中以某种方式保护代码被称为[线程安全][Thread Safety]

>数据共享是多线程编程之所以复杂且容易潜藏错误的主因. Although often essential, it pays to keep it as simple as possible.

被阻塞的线程不占用CPU资源.

## [Join and Sleep](http://blog.chenxu.me/post/detail?id=f5add332-8f8d-4456-aea4-a411c054a4bd)
通过调用 __Join__ 方法使得当前线程等待另一个线程结束. 如:

```csharp
static void Main()
{
  Thread t = new Thread (Go);
  t.Start();
  t.Join();
  Console.WriteLine ("Thread t has ended!");
}
 
static void Go()
{
  for (int i = 0; i < 1000; i++) Console.Write ("y");
}
```

首先打印"y"1000次, 然后立即打印"Thread t has ended!". 可以在调用 __Join__ 时指定超时时间, 可以是毫秒也可以是 __TimeSpan__ . 如果返回 __true__ 表示线程正常结束, 如果返回 __false__ 表示超时.

__Thread.Sleep__ 让当前线程暂停指定周期:

```csharp
Thread.Sleep (TimeSpan.FromHours (1));  // sleep for 1 hour
Thread.Sleep (500);                     // sleep for 500 milliseconds
```

调用 __Sleep__ 或 __Join_ _后, 线程被阻塞且不再占用CPU资源.

> __Thread.Sleep(0)__ 让当前线程立即放弃其占用的时间片, 自愿交出CPU给其它线程(包括其它CPU内核上的线程). Framework 4.0提供的新 __Thread.Yield()__ 方法也做类似的事情, 但限定当前CPU内核上的线程.
>
> __Sleep(0)__ or __Yield__ is occasionally useful in production code for advanced performance tweaks. It’s also an excellent diagnostic tool for helping to uncover thread safety issues: if inserting Thread.Yield() anywhere in your code makes or breaks the program, you almost certainly have a bug.

## How Threading Works

多线程由线程调度器(一个由CLP委托给操作系统的函数)管理, 线程调度器确保所有活动线程都会分配到合适的执行时间, 等待中(如等待独占锁)或被阻塞(如等待用户输入)的线程不占用CPU时间.

对于单核电脑, 线程调度器实行 _time-slicing_ 机制: rapidly switching execution between each of the active threads. 在Windows下, 一个time-slice一般在10毫秒左右, 远大于CPU切换线程上下文的时间(一般在几微秒).

对于多核电脑, multithreading is implemented with a mixture of time-slicing and genuine concurrency, where different threads run code simultaneously on different CPUs. It’s almost certain there will still be some time-slicing, because of the operating system’s need to service its own threads — as well as those of other applications.

A thread is said to be preempted when its execution is interrupted due to an external factor such as time-slicing. In most situations, a thread has no control over when and where it’s preempted.

## Threads vs Processes

TODO

## Threading’s Uses and Misuses

TODO

# __Creating and Starting Threads__

如简介里看到的那样, 线程使用 __Thread__ 类创建, 传递 __ThreadStart__ 委托指定执行入口. 下面给出了 __ThreadStart__ 委托的定义:

````csharp
public delegate void ThreadStart();
````

调用 __Start__ 让线程进入就绪状态. The thread continues until its method returns, at which point the thread ends. 下面的例子使用C#扩展语法创建 __ThreadStart__ 委托:

```csharp
class ThreadTest
{
  static void Main() 
  {
    Thread t = new Thread (new ThreadStart (Go));
 
    t.Start();   // Run Go() on the new thread.
    Go();        // Simultaneously run Go() in the main thread.
  }
 
  static void Go()
  {
    Console.WriteLine ("hello!");
  }
}
```

例子里, 线程 __t__ 执行 __Go()__ , 与此同时 _main_ 线程也调用 __Go()__ . 结果就是几乎同时打印的两个hello.

直接传递一个方法(由C#自行推断 __ThreadStart__ 委托)可以理方便的创建线程:

```csharp
Thread t = new Thread (Go);    // No need to explicitly use ThreadStart
```

另一个快捷方法是使用lambda表达式或匿名方法:

```csharp
static void Main()
{
  Thread t = new Thread ( () => Console.WriteLine ("Hello!") );
  t.Start();
}
```

## Passing Data to a Thread

向线程方法传递参数的最简捷方式就是使用lambda表达式间接调用目标方法:

```csharp
static void Main()
{
  Thread t = new Thread ( () => Print ("Hello from t!") );
  t.Start();
}
 
static void Print (string message) 
{
  Console.WriteLine (message);
}
```

通过这个方式, 你可以向线程方法传递任意数量的参数. You can even wrap the entire implementation in a multi-statement lambda:

```csharp
new Thread (() =>
{
  Console.WriteLine ("I'm running on another thread!");
  Console.WriteLine ("This is so easy!");
}).Start();
```

C# 2.0后你也可以通过匿名方法实现同样的效果:

```csharp
new Thread (delegate()
{
  ...
}).Start();
```

__Thread.Start__ 也可以向线程传参:

```csharp
static void Main()
{
  Thread t = new Thread (Print);
  t.Start ("Hello from t!");
}
 
static void Print (object messageObj)
{
  string message = (string) messageObj;   // We need to cast here
  Console.WriteLine (message);
}
```

这个方法可行的秘密在于 __Thread__ 的两个重载接受两个委托的构造函数:

```csharp
public delegate void ThreadStart();
public delegate void ParameterizedThreadStart (object obj);

```

__ParameterizedThreadStart__ 的不足之处它只接受一个 __object__ 类型的参数, 你需要类型转换.

**_Lambda expressions and captured variables_**

正如我们看到的, lambda表达式是一个向线程传参的强大方式. 然而你要小心在启动线程后意外修改 _captured variables_ , 因为这些是共享变量. 考虑下面的代码:

```csharp
for (int i = 0; i < 10; i++)
  new Thread (() => Console.Write (i)).Start();
```

输出是不确定的,下面是一个可能的结果:

```shell
#output
0223557799
```

问题在于在循环的生命期间变量 __i__ 指向相同的内存地址. 因此每个线程调用 __Console.Write__ 输出变量时, 其值有可能被改变.

>This is analogous to the problem we describe in “Captured Variables” in Chapter 8 of C# 4.0 in a Nutshell. The problem is less about multithreading and more about C#'s rules for capturing variables (which are somewhat undesirable in the case of __for__ and __foreach__ loops).

解决方案是使用临时变量, 如:

```csharp
for (int i = 0; i < 10; i++)
{
  int temp = i;
  new Thread (() => Console.Write (temp)).Start();
}
```

现在变量 __temp__ 属于每次迭代循环. 因此各个线程使用不同的内存地址也就没问题了. 下面是另一个能更容易阐明问题的例子:

```csharp
string text = "t1";
Thread t1 = new Thread ( () => Console.WriteLine (text) );
 
text = "t2";
Thread t2 = new Thread ( () => Console.WriteLine (text) );
 
t1.Start();
t2.Start();
```

```shell
#output
t2
t2
```

## Naming Threads
每个线程都有个你可以设置的 __Name__ 属性, 这对调试有好处. 特别是在使用Visual Studio时, 因为线程名字将显示在线程窗口和调试工具栏. 你只能给线程设置一个名字, 重复操作将抛出异常.

静态属性 __Thread.CurrentThread__ 能让你得到当前运行中的线程. 下面的例子展示你如何给 _main_ 线程设置名字:

```csharp
class ThreadNaming
{
  static void Main()
  {
    Thread.CurrentThread.Name = "main";
    Thread worker = new Thread (Go);
    worker.Name = "worker";
    worker.Start();
    Go();
  }
 
  static void Go()
  {
    Console.WriteLine ("Hello from " + Thread.CurrentThread.Name);
  }
}
```

## Foreground and Background Threads

默认情况下, 你显式创建的线程都是 _foreground_ 线程. 前台线程只要还在运行状态就能维持应用程序的运行状态, 而 _background_ 线程没有这项能力. 一旦全部前台线程运行完毕, 应用程序同时停止运行, 而仍然在运行当中的后台线程也会被强制停止.

> 线程是前台线程还是后台线程与它的优先级跟分配的时间片没直接关系.

你可以通过 __IsBackground__ 属性获取或修改线程的这一状态. 下面是例子:

```csharp
class PriorityTest
{
  static void Main (string[] args)
  {
    Thread worker = new Thread ( () => Console.ReadLine() );
    if (args.Length > 0) worker.IsBackground = true;
    worker.Start();
  }
}
```

如果程序是无参启动, 工作线程默认为前台线程并一直等待并接收用户输入直到用户输入Enter. 与此同时, 主线程虽然已经退出运行, 但因为仍然前台线程处于运行状态所以应用程序依然保持运行状态.

如果程序是带参启动, 工作线程被置为后台线程, 一旦主线程结束, 程序就会同时结束( __ReadLine__ 被强制停止运行).

当进程以这种方式停止工作, 所有后台线程的 __finally__ 代码块都将失去运行机会. 如果你的程序使用 __finally__ 或 __using__ 块进行诸如释放资源或删除临时文件等清理工作, 这将是个问题. 为了避免这个情况, 你可以显式的让应用程序在所有后台线程停止运行后再结束. 有两个方法可以达到这个目的:

* 对于那些你自己创建的线程, 调用线程的 __Join__ 方法
* 对于那些线程池创建的线程, 使用[event wait handle][Signaling with Event Wait Handles]

对于这两种方式, 你都应该指定一个超时时间, 这样你就无需持续跟踪线程,仅仅是因为担心它无法正常结束. 这是你的补救退出机制: 你希望你的应用程序都最终正常关闭, 而不是打开任务管理器杀进程!

> If a user uses the Task Manager to forcibly end a .NET process, all threads “drop dead” as though they were background threads. This is observed rather than documented behavior, and it could vary depending on the CLR and operating system version.

前台线程无需特殊处理, 但你要小心避免那些会导致线程无法正常结束的bugs. 应用程序无法正常退出一般是因为活动的前台线程.

## Thread Priority

线程的 __Priority__ 属性决定了它相对于其它线程, 能从操作系统中获取多少执行时间, 有以下几种比例:

```csharp
enum ThreadPriority { Lowest, BelowNormal, Normal, AboveNormal, Highest }
```

只有当多个线程同时处于活动状态时这个属性才有实际意义.

> 要谨慎提升线程的优先级 - 因为这可能会导致其它线程完全失去就有的资源.

提升线程的优先级不会让它变得更适合执行实时工作, 因为它依然受到应用程序的进程优先级所限制. 为了执行实时工作, 必须同时提升进程的优先级, 你可以使用 __System.Diagnostics__ 命名空间下的 __Process__ 进行修改(我们不会告诉你如何做):

```csharp
using (Process p = Process.GetCurrentProcess())
  p.PriorityClass = ProcessPriorityClass.High;
```

实际上, __ProcessPriorityClass.High__ 的优先级比最高优先级: __Realtime__ 要低一档. 把进程的优先级设置为 __Realtime__ 实际上就是告诉OS, 你永远不希望进程让出CPU时间给其它进程. 如果你的程序意外进入死循环, 你的的操作系统将会被锁死, 除了按下电源开关你别无它法. 因此, 通常情况下 __High__ 就是实时应用程序的最佳选择.

> If your real-time application has a user interface, elevating the process priority gives screen updates excessive CPU time, slowing down the entire computer (particularly if the UI is complex). Lowering the main thread’s priority in conjunction with raising the process’s priority ensures that the real-time thread doesn’t get preempted by screen redraws, but doesn’t solve the problem of starving other applications of CPU time, because the operating system will still allocate disproportionate resources to the process as a whole. An ideal solution is to have the real-time worker and user interface run as separate applications with different process priorities, communicating via Remoting or memory-mapped files. Memory-mapped files are ideally suited to this task; we explain how they work in Chapters 14 and 25 of C# 4.0 in a Nutshell.

尽管可以提升进程优先级, 但在托管环境下处理那些对实时性要求比较高的工作时还是有不小的限制. 另外自动垃圾回收也会导致一些潜在问题, the operating system may present additional challenges — even for unmanaged applications — that are best solved with dedicated hardware or a specialized real-time platform.

## Exception Handling
在 __try/catch/finally__ 代码块里创建的线程一旦开始执行, 都将与这些代码块断开关联. 考虑下面的程序:

```csharp
public static void Main()
{
  try
  {
    new Thread (Go).Start();
  }
  catch (Exception ex)
  {
    // We'll never get here!
    Console.WriteLine ("Exception!");
  }
}
```

上面的 __try/catch__ 语句是无效的, 新建的线程将会被未处理异常 __NullReferenceException__ 所阻塞. 如果你回想起前面所说的每个线程都有独立的执行路径就能理解这个行为是合理的.

纠正方法就是把异常处理代码移到 __Go__ 方法中:

```csharp
public static void Main()
{
   new Thread (Go).Start();
}
 
static void Go()
{
  try
  {
    // ...
    throw null;    // The NullReferenceException will get caught below
    // ...
  }
  catch (Exception ex)
  {
    // Typically log the exception, and/or signal another thread
    // that we've come unstuck
    // ...
  }
}
```

对于要交付客户的应用程序, 你需要在所有线程的入口方法里加上异常处理代码 - 一如你在主线程上做的那样. 未处理异常会让整个应用程序崩溃. 哦, 还有一个让人沮丧的弹窗!

> In writing such exception handling blocks, rarely would you ignore the error: typically, you’d log the details of the exception, and then perhaps display a dialog allowing the user to automatically submit those details to your web server. You then might shut down the application — because it’s possible that the error corrupted the program’s state. However, the cost of doing so is that the user will lose his recent work — open documents, for instance.

> The “global” exception handling events for WPF and Windows Forms applications (Application.DispatcherUnhandledException and Application.ThreadException) fire only for exceptions thrown on the main UI thread. You still must handle exceptions on worker threads manually.
>
> AppDomain.CurrentDomain.UnhandledException fires on any unhandled exception, but provides no means of preventing the application from shutting down afterward.

不过还是有些情况下, 工作线程上的异常无需你处理, 因为.Net Framework已经为你处理了. These are covered in upcoming sections, and are:

* [Asynchronous delegates](#Asynchronous+delegates)
* [BackgroundWorker][BackgroundWorker]
* The [Task Parallel Library][The Parallel Class] (conditions apply)

# __Thread Pooling__
每当你启动一个线程, 都要花费几百微秒用于初始化工作, 如开辟新的变量栈. 另外默认参数下每个线程消耗1MB的内存. _thread pool_ 通过共享和重复利用线程来减少这些开支, allowing multithreading to be applied at a very granular level without a performance penalty. This is useful when leveraging multicore processors to execute computationally intensive code in parallel in “divide-and-conquer” style.














**_Asynchronous delegates_**<a name="Asynchronous+delegates" />

[Source Link]: http://www.albahari.com/threading
[NewThread.png]: /assets/Threading_In_CSharp_Part_1/NewThread.png
[Locking]: ./Threading-In-CSharp-Part-2.md#Locking
[Blocking]: ./Threading-In-CSharp-Part-2.md#Blocking
[Thread Safety]: ./Threading-In-CSharp-Part-2.md#Thread+Safety
[Signaling with Event Wait Handles]: ./Threading-In-CSharp-Part-2.md#Signaling+with+Event+Wait+Handles
[BackgroundWorker]: ./Threading-In-CSharp-Part-3.md#BackgroundWorker
[The Parallel Class]: ./Threading-In-CSharp-Part-5.md#The+Parallel+Class
