---
layout: post
---

# Chapter 4. 理解所有权(Understanding Ownership)

所有权是Rust最独特的特征，它使得Rust无须垃圾回收器也能保证内存安全。因此理解所有权在Rust中的工作原理相当重要。在本章我们会介绍所有权以及一些相关特征：借用，切片以及Rust数据的内存布局。

## 什么是所有权(What Is Ownership?)

所有权是Rust的主要特征，解释起来相当简单，但它对Rust有着深远的影响。

所有的程序都必须管理它们在运行时使用计算机内存的方式。一些语言具有垃圾回收器，在程序运行时不断寻找不再使用的内存并回收。而另一些语言则需要程序员明确地分配与释放内存。Rust使用第三种策略：内存受所有权系统管理，所有权系统包含一些规则，编译器在编译期检查这些规则。所有权特征不会降低程序的运行速度。

对一些程序员而言所有权是个新概念，需要一些时间习惯。好消息是，当你彻底掌握Rust与所有权系统的相关规则，你就能自然地开发出安全而高效的代码。坚持住。

当你理解了所有权，你就有了一个坚实的基础去了解那些使Rust独特的特征。在本章，我们通过一些例子来学习所有权，这些例子使用了一个非常通用的数据结构：字符串。

> ### 栈和堆(The Stack and the Heap)
> 在许多编程语言中，你无须经常考虑栈和堆。但在像Rust这样的系统编程语言中，值是在栈还是在堆上，深刻的影响着语言的行为方式以及你的某些决定。Parts of ownership will be described in relation to the stack and the heap later in this chapter, 所以在这里只作一个简短的解释。
> 
> 栈和堆作为内存的一部分，供你的代码在运行时使用，但它们的构造方式有所不同。栈按照获取值的顺序存储值，并以相反的顺序删除值。这也被称为后进先出。想像一个盘子栈：当你要加入多个盘子时，你把它们挨个放在栈的顶部，而当你需要盘子时，会从栈的顶部取。你不能从中间或底部存取盘子。加入数据被称为压入到栈，移除数据被称为从栈弹出。
> 
> 存储到栈的数据必须要有一个已知的固定长度。那些在编译时无法确定大小或可能在运行时改变大小的数据必须存储到堆上。堆缺少组织：当你要把数据放到堆上，你请求一定大小的空间。操作系统在堆中某处找了个足够大的空间，把它标记为使用中，并返回一个指针（空间的地址）。此过程被称为在堆上进行分配，有时也简称“分配”。把数值压入到栈不叫分配。因为指针是已知且长度固定的，所以你可以把指针存储到栈上，当你要获取实际数据时，你必须要对指针解引用。
> 
> 想像下你与朋友去餐厅就餐。当你们去到餐厅，你把你们的人数告诉服务员，服务员找一张有足够位置餐桌并把你们领过去。如果有朋友迟到，他可以询问你们所在的餐桌位置最终找到你们。
> 
> 数据入栈比在堆上分配空间要快，因为入栈无须搜索用于存储新数据的空间。因为空间永远在栈顶。相比之下，在堆上分配空间要做更多的工作，首先操作系统要找到一个足够大的空间来存储数据，然后执行一些登记工作以便下次分配能正常进行。
> 
> 访问堆中的数据也要比访问栈上的数据要慢，因为你要对指针进行解引用。现代处理器在处理内存地址的短距离跳转时有更快的响应速度。继续之前的比喻，餐厅服务员要给几张桌下单。他给一张桌下完单再去下一张桌下单是最高效的。而在一张桌下部分单又去另一张桌下部分单再回到前一张桌下单，如此重复一定是最低效的。基于相同的理由，处理器处理距离较近的数据（在栈上）比处理距离较远的数据（在堆上）要快。另外在堆上分配足够大的空间也是件费时的工作。
> 
> 当你的代码调用一个函数，要传入函数的值（包含可能是指向在堆上的数据的指针）与函数本身的本地变量被压入栈。当函数调用结束，这些值被弹出栈。
> 
> 持续跟踪哪部分代码正在使用堆上的哪部分数据，减少堆上的重复数据，及时清除堆上不再使用的数据避免你遇到空间不足的提示，就是所有权所涉及的全部问题。一旦你理解了所有权，你就不必经常考虑栈和堆，but knowing that managing heap data is why ownership exists can help explain why it works the way it does.

## 所有权规则(Ownership Rules)

我们先看看所有权规则。把这些规则牢牢记住，我们会通过一些例子来解释它们：

* Rust中的每个值都有一个被称为 *所有者(ower)* 的变量。
* 同一时间只能有一个 *所有者(ower)*
* 当 *所有者(ower)* 离开 *作用域(scope)* ，相应的值也会被删除。

## 变量的作用域(Variable Scope)

我们已经在第2章中介绍过Rust的程序示例。从现在开始，我们跳过基础语法，我们不再在示例中包含类似 `fn main() {` 这样的代码，所以如果你要运行示例代码，你要手动的加上 `main` 函数。这样我们的示例代码就能理简洁一些，我们也可以更专注于实际细节而不是那些样板代码。

作为第一个所有权的示例，我们将会看看一些变量的作用域。作用域就是一个对象在程序里的有效范围。假如我们有以下一个变量：

```rust
let s = "hello";
```

变量 `s` 指向一个字符串字面值，其值被硬编码到我们的程序源代码中。从这个变量被声明处开始，一直到当前作用域结束，它都是有效的。下面的注释能更直观的解释：

```rust
{                   // s在此处无效，因为它还没被声明
    let s = "hello";// s从此处开始有效

    //可以对s进行有效操作
}                   // 作用域到此结束，s不再有效
```

换句话说，有两个重要的时间点：

* 当 `s` 进入作用域，它就有效
* 在离开作用域前，它都是有效的

在这一点上，作用域跟变量何时有效的关系与其它编程语言是相似的。现在我们将通过引入 `String` 类型来构建这种理解。

## `String` 类型

为了说明所有权的相关规则，我们需要一种比在第3章介绍过的复杂一点的数据类型。在此之前使用过的类型全是存储在栈上且在离开作用域后被弹出栈，而我们需要查看存储在堆上的数据，探索Rust如何知道何时清除相关数据。

在这里我们会使用 `String` 作为示例，专注于 `String` 与所有权相关的那部分内容。