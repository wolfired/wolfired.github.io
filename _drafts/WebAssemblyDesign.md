---
layout: post
---

<!-- TOC -->

- [WebAssembly High-Level Goals - WebAssembly高级目标](#webassembly-high-level-goals---webassembly高级目标)
- [JavaScript API](#javascript-api)
    - [Traps - 陷阱](#traps---陷阱)

<!-- /TOC -->

# WebAssembly High-Level Goals - WebAssembly高级目标

1. 定义一种可移植

2. 渐进提升和增量实现

    * 基于标准的[最小可行产品(MVP)](https://webassembly.org/docs/mvp/)，有着与[asm.js](http://asmjs.org/)类似的功能，主要针对[C/C++](https://webassembly.org/docs/c-and-c++/)

    * 首先专注于实现如[线程](https://github.com/WebAssembly/design/issues/1073)、[零消耗异常](https://github.com/WebAssembly/design/issues/1078)、[SIMD](https://github.com/WebAssembly/design/issues/1075)等附加功能，然后依据体验和反馈加入其它附加功能，如支持C/C++以外的编程语言。

3. 可与现有[Web平台](https://webassembly.org/docs/web/)完美整合并运行：

    * 
    
    * 在Javascript的语义下运行；

    * 与Javascript双向同步调用；

    * 强制执行同源和权限安全策略；

    * 与Javascript相同的Web APIs访问浏览器功能；

    * 定义一种可以与二进制格式双向转换的可编辑文本格式，技能查看代码功能。

4. 可嵌入到非浏览器运行环境

5. 打造一个伟大的平台：

    * 为WebAssembly新建一个LLVM后端，同时移植到clang([why LLVM first?](https://webassembly.org/docs/faq/#which-compilers-can-i-use-to-build-webassembly-programs))；

    * 推广针对WebAssembly的其它编译器和工具；

    * 带来更多实用工具

# JavaScript API

在当前的[最小可行产品(MVP)](https://webassembly.org/docs/mvp/)中想要在Web平台上访问WebAssembly只能通过Javascript API。而在将来，WebAssembly可由HTML标签`<script type='module'>`或者其它能通过URL加载ES6模块的Web API（是[ES6模块集成](https://webassembly.org/docs/modules/#integration-with-es6-modules)的一部分）加载并直接运行。

## Traps - 陷阱

