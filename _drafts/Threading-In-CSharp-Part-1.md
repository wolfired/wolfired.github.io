---
layout: post
---

[TOC]

# __[PART 1: GETTING STARTED](http://www.albahari.com/threading/)__

## Introduction and Concepts

C#支持通过多线程并行执行代码, 一个线程就是一个独立的执行路径, 与其它线程同时运行.

一个C# _client_ 程序(Console程序, WPF程序或Windows Forms程序)开始于一个由CLR和OS自动创建的线程(_main_ 线程), 通过创建附加线程实现多线程运行. 下面是一个简单例子:

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

```csharp
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
```
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

```
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
```
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

当两个线程同时抢夺同一把锁(在这里是__locker__), 在锁被释放前, 其中一个线程将会等待或[阻塞][Blocking]. 在这个例子里, 锁确保了在同一时刻只有一个线程可以进入临界代码, 而"Done"只会打印一次. 在多线程的不确定状态中以某种方式保护代码被称为[线程安全][Thread Safety]

>数据共享是多线程编程之所以复杂且容易潜藏错误的主因. Although often essential, it pays to keep it as simple as possible.

被阻塞的线程不占用CPU资源.

## [Join and Sleep](http://blog.chenxu.me/post/detail?id=f5add332-8f8d-4456-aea4-a411c054a4bd)
通过调用__Join__方法使得当前线程等待另一个线程结束. 如:

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

首先打印"y"1000次, 然后立即打印"Thread t has ended!". 可以在调用__Join__时指定超时时间, 可以是毫秒也可以是__TimeSpan__. 如果返回__true__表示线程正常结束, 如果返回__false__表示超时.

__Thread.Sleep__让当前线程暂停指定周期:

```csharp
Thread.Sleep (TimeSpan.FromHours (1));  // sleep for 1 hour
Thread.Sleep (500);                     // sleep for 500 milliseconds
```

调用__Sleep__或__Join__后, 线程被阻塞且不再占用CPU资源.

>__Thread.Sleep(0)__让当前线程立即放弃其占用的时间片, 自愿交出CPU给其它线程(包括其它CPU内核上的线程). Framework 4.0提供的新__Thread.Yield()__方法也做类似的事情, 但限定当前CPU内核上的线程.
>
>__Sleep(0)__ or __Yield__ is occasionally useful in production code for advanced performance tweaks. It’s also an excellent diagnostic tool for helping to uncover thread safety issues: if inserting Thread.Yield() anywhere in your code makes or breaks the program, you almost certainly have a bug.

## How Threading Works

多线程由线程调度器(一个由CLP委托给操作系统的函数)管理, 线程调度器确保所有活动线程都会分配到合适的执行时间, 等待中(如等待独占锁)或被阻塞(如等待用户输入)的线程不占用CPU时间.

对于单核电脑, 线程调度器实行_time-slicing_机制: rapidly switching execution between each of the active threads. 在Windows下, 一个time-slice一般在10毫秒左右, 远大于CPU切换线程上下文的时间(一般在几微秒).

对于多核电脑, multithreading is implemented with a mixture of time-slicing and genuine concurrency, where different threads run code simultaneously on different CPUs. It’s almost certain there will still be some time-slicing, because of the operating system’s need to service its own threads — as well as those of other applications.

A thread is said to be preempted when its execution is interrupted due to an external factor such as time-slicing. In most situations, a thread has no control over when and where it’s preempted.

## Threads vs Processes

TODO

## Threading’s Uses and Misuses

TODO

# Creating and Starting Threads

如简介里看到的那样, 线程使用__Thread__类创建, 传递__ThreadStart__委托指定执行入口. 下面给出了__ThreadStart__委托的定义:

````csharp
public delegate void ThreadStart();
````

调用__Start__让线程进入就绪状态. The thread continues until its method returns, at which point the thread ends. 下面的例子使用C#扩展语法创建__ThreadStart__委托:

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

例子里, 线程__t__执行__Go()__, 与此同时_main_线程也调用__Go()__. 结果就是几乎同时打印的两个hello.

直接传递一个方法(由C#自行推断__ThreadStart__委托)可以理方便的创建线程:

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


[NewThread.png]: /assets/Threading_In_CSharp_Part_1/NewThread.png
[Locking]: ./Threading-In-CSharp-Part-2.md#Locking
[Blocking]: ./Threading-In-CSharp-Part-2.md#Blocking
[Thread Safety]: ./Threading-In-CSharp-Part-2.md#Thread+Safety
