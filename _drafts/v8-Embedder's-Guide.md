---
layout: post
---

# [Embedder's Guide][Embedder's Guide]

在通读[入门指南][Getting Started]后你想必对使用V8作为一个独立的虚拟机以及V8的一些关键概念如：句柄、作用域、上下文有了一定的了解。本文章进一步探讨这些概念并引入其它对把V8嵌入到你的C++应用起着关键作用的概念。

V8 API提供了用于：编译与运行脚本、访问C++方法与数据结构、处理错误、安全检查的函数。你的应用可以像使用其它C++库那样使用V8库。你的C++代码通过引入`include/v8.h`调用V8 API访问V8库。

当你要针对V8优化你的应用时，[V8设计元素][Design Elements]为你提供了有用的背景资料。

## 读者

本文档的目标读者是那些要把V8引擎嵌入到C++应用的C++程序员。它帮助你把C++应用的对象和方法暴露给Javascript，也把Javascript的对象和函数暴露给C++应用。

## 句柄和垃圾回收

句柄代表着Javascript对象在堆中位置的引用。V8垃圾回收器回收并再利用那些已经无法再被访问的对象占用的内存。在垃圾回收过程中，回收器经常移动对象在堆中的位置。当回收器移动了一个对象，它也把对象的新位置更新到所有指向这个对象的句柄。

当一个对象无法从Javascript访问且已经没有句柄指向它时，它就被标记为垃圾。垃圾回收器不时的移除所有被标记为垃圾的对象。V8的回收机制是V8的性能关键。要想了解更多，可参考[V8设计元素][Design Elements]。

有多种类型的句柄：

* 本地句柄保存在栈上，调用适当的析构器就能删除。这类句柄的生命周期由句柄作用域决定，作用域通常在函数调用初期创建。删除作用域后，垃圾回收器可以自由的释放作用域内句柄指向的对象，前提是这些对象已经无法从Javascript访问且已经没有其它句柄指向它们。在[入门指南][Getting Started]中就使用到这种类型的句柄。

本地句柄的类如：`Local<SomeType>`。

> 注意：句柄栈不是C++调用栈的一部分，但句柄作用域被嵌入到C++调用栈。句柄作用域只能在栈上分配，而不能用new创建。

* 持久化句柄指向了堆上分配的Javascrip对象，正如本地句柄指向栈上分配的Javascript对象。两者的不同这处在于生命周期的管理方式。当你要保存一个要在多个函数调用中使用的对象的引用，又或者句柄的生命周期与C++作用域不一致时，请使用持久化句柄。如Google Chrome就使用持久化句柄保存DOM节点。可以通过调用`PersistentBase::SetWeak`把一个持久化句柄变成弱句柄，当一个对象只被一个弱持久化句柄引用时会让垃圾回收器触发一个回调。

    * `UniquePersistent<SomeType>`依靠底层对象的构造器和析构器管理生命周期。

    * `Persistent<SomeType>`可以由它的构造器构造，但必须显式的由`Persistent::Reset`清除。

* 另外还有一些很少用到的句柄类型，在这里我们只简单提一下：

    * `Eternal`作为一种持久化句柄，用于那些期望永不删除的Javascript对象。它的使用成本很低，因为它不需要垃圾回收器反复确定对象的存活性。

    * `Persistent`跟`UniquePersistent`都是不能复制的，这使得它们不能作为C++ 11之前版本标准库容器的值。`PersistentValueMap`跟`PersistentValueVector`作为替代方案使用。C++ 11后的版本就不需要使用替代方案了。

每创建一个对象都创建一个本地句柄会导致大量的句柄被创建。这时句柄作用域的优势就出来了。你可以把句柄作用域想像成一个持有大量句柄的句柄容器。调用句柄作用域的析构器后，作用域内的所有句柄都会从栈上删除。正如你期望的那样，这会使得垃圾回收器把这些句柄指向的对象从堆上删除。

回到我们的例子[入门指南][Getting Started]，从下图中你可以看到句柄栈和堆分配对象。注意`Context::New()`返回一个本地句柄，基于它我们创建了一个持久化句柄用于演示。

![][local_persist_handles_review]

调用析构器`HandleScope::~HandleScope`后句柄作用域被删除。作用域内的句柄所指向的对象被标记为可移除，在下轮垃圾回收时，如果这些对象已经没有被引用，那就会被真正移除。垃圾回收器同样可以移除在堆上的对象`source_obj`和`script_obj`，只要它们没有被引用或已经无法从Javascript中访问。由于上下文句柄属于持久化句柄，在离开句柄作用域后它不会被移除。只有显式的调用`Reset`才能移除上下文句柄。

> 注意：贯穿本文档，句柄专指本地句柄，当涉及到持久化句柄时会使用全称。

另外：你不能直接返回一个函数内的句柄作用域内的本地句柄。因为在函数返回前，句柄作用域就会被移除，其内部的本地句柄也会被移除。正确的做法是创建`EscapableHandleScope`取代`HandleScope`，把你要返回的本地句柄作为参数调用`Escape`。现在是一段演示代码：
```c++
// This function returns a new array with three elements, x, y, and z.
Local<Array> NewPointArray(int x, int y, int z) {
  v8::Isolate* isolate = v8::Isolate::GetCurrent();

  // We will be creating temporary handles so we use a handle scope.
  EscapableHandleScope handle_scope(isolate);

  // Create a new empty array.
  Local<Array> array = Array::New(isolate, 3);

  // Return an empty result if there was an error creating the array.
  if (array.IsEmpty())
    return Local<Array>();

  // Fill out the values
  array->Set(0, Integer::New(isolate, x));
  array->Set(1, Integer::New(isolate, y));
  array->Set(2, Integer::New(isolate, z));

  // Return the value through Escape.
  return handle_scope.Escape(array);
}
```

## 上下文

在V8里，上下文是在一个V8实例上运行隔离的不相关的Javascript应用的执行环境。要运行Javascript代码都需要你显式的指定上下文。

为什么需要上下文？因为Javascript提供了很多可被Javascript代码修改的内建工具函数和对象。如果两个不相关的Javascript函数修改了同一个全局对象会导致无法预料的结果。

从CPU时间和内存的角度看，创建一个带必要内建对象的执行上下文消耗巨大。除了第一个创建的上下文消耗比较大，V8的缓存机制确保了后续创建的上下文相对快捷。虽然你创建的首个上下文有些昂贵，但透过V8的缓存机制可以确保后续创建相对便宜。这是因为首个被创建的上下文需要构造内建对象和解析内部Javascript代码，而后续上下文只需要给自己构造内建对象。通过V8的快照特性(通过默认编译选项`snapshot=yes`激活)，可以大大压缩创建首个上下文所消耗的时间，因为快照是一个包含已编译内部Javascript代码的序列化堆。除了垃圾回收机制，V8的缓存机制同样是V8的性能关键，可参考[V8设计元素][Design Elements]。

在你创建上下文后，你可以重复进入或退出。当在进入上下文A后，你同样可以进入上下文B，这意味着你把当前上下文从A切换成B，而在你退出B后，当前上下文将重新切换回A。如下图所示：

![][intro_contexts]

要注意的是每个上下文的内建工具函数和对象都是独立的。在你创建上下文时，你可以设置一个安全令牌。参考`Security Model`获取更多信息。

在V8中使用上下文的目的是在浏览器中，每个window和iframe可以有一个私有的Javascript环境。

## 模板

模板是上下文中Javascript函数和对象的蓝图。你可以使用模板将C++函数和数据结构封装成Javascript对象，这样你就可以通过Javascript代码操纵这些函数和数据结构。例如，Google Chrome使用模板把C++ DOM节点封装成Javascript对象，还把C++函数绑定到global命名空间。你可以创建很多模板，而你创建的所有上下文都可以使用这些模板。只要你有需要就可以创建模板，模板数量没有限制。但一个上下文只能有一个模板实例。

在Javascript里，函数跟对象有着巨大的差异。在Java或C++里，要创建一个新类型的对象，你需要定义一个新的类。而在Javascript里，你定义一个函数，把这个函数作为构造函数创建实例。Javascript对象的布局与功能与构造它的函数紧密关联。








[Embedder's Guide]: https://github.com/v8/v8/wiki/Embedder's-Guide
[Getting Started]: https://github.com/v8/v8/wiki/Getting-Started-with-Embedding
[Design elements]: https://github.com/v8/v8/wiki/Design-Elements

[local_persist_handles_review]: /assets/v8_Embedders_Guide/local_persist_handles_review.png

[intro_contexts]: /assets/v8_Embedders_Guide/intro_contexts.png