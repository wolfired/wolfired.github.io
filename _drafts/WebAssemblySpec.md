---
layout: post
---

<!-- TOC -->

- [__Introduction - 引言__](#__introduction---引言__)
    - [_Introduction - 简介_](#_introduction---简介_)
        - [Design Goals - 设计目标](#design-goals---设计目标)
        - [Scope](#scope)
        - [Dependencies - 依赖](#dependencies---依赖)
    - [_Overview - 概览_](#_overview---概览_)
        - [Concepts - 概念](#concepts---概念)
        - [Semantic Phases - 语言阶段](#semantic-phases---语言阶段)
- [__Structure - 结构__](#__structure---结构__)
    - [Conventions - 约定](#conventions---约定)
        - [Grammar Notation - 语法符号](#grammar-notation---语法符号)
        - [Auxiliary Notation - 辅助符号](#auxiliary-notation---辅助符号)
        - [Vectors](#vectors)
    - [Values - 值](#values---值)
        - [Bytes - 字节](#bytes---字节)
        - [Integers - 整数](#integers---整数)
        - [Floating-Point - 浮点数](#floating-point---浮点数)
        - [Names](#names)
    - [Types - 类型](#types---类型)
        - [Value Types - 值类型](#value-types---值类型)
        - [Result Types - 结果类型](#result-types---结果类型)
        - [Function Types - 函数类型](#function-types---函数类型)
        - [Limits](#limits)
        - [Memory Types - 内存类型](#memory-types---内存类型)
        - [Table Types - 表类型](#table-types---表类型)
        - [Global Types](#global-types)
        - [External Types](#external-types)
    - [Instructions - 指令](#instructions---指令)
        - [Numeric Instructions](#numeric-instructions)
        - [Parametric Instructions](#parametric-instructions)
        - [Variable Instructions](#variable-instructions)
        - [Memory Instructions](#memory-instructions)
        - [Control Instructions](#control-instructions)
        - [Expressions](#expressions)
    - [Modules - 模块](#modules---模块)
        - [Indices - 索引](#indices---索引)
        - [Types - 类型](#types---类型-1)
        - [Functions - 函数](#functions---函数)
        - [Tables - 表](#tables---表)
        - [Memories - 内存](#memories---内存)
        - [Globals](#globals)
        - [Element Segments](#element-segments)
        - [Data Segments - 数据段](#data-segments---数据段)
        - [Start Function - 开始函数](#start-function---开始函数)
        - [Exports - 导出](#exports---导出)
        - [Imports - 导入](#imports---导入)
- [__Validation - 验证__](#__validation---验证__)
    - [Conventions](#conventions)
        - [Contexts](#contexts)
        - [Prose Notation](#prose-notation)
        - [Formal Notation](#formal-notation)
    - [Types](#types)
        - [Limits](#limits-1)
        - [Function Types](#function-types)
        - [Table Types](#table-types)
        - [Memory Types](#memory-types)
        - [Global Types](#global-types-1)
    - [Instructions](#instructions)
        - [Numeric Instructions](#numeric-instructions-1)
        - [Parametric Instructions](#parametric-instructions-1)
        - [Variable Instructions](#variable-instructions-1)
        - [Memory Instructions](#memory-instructions-1)
        - [Control Instructions](#control-instructions-1)
        - [Instruction Sequences](#instruction-sequences)
        - [Expressions](#expressions-1)
    - [Modules](#modules)
        - [Functions](#functions)
        - [Tables](#tables)
        - [Memories](#memories)
        - [Globals](#globals-1)
        - [Element Segments](#element-segments-1)
        - [Data Segments](#data-segments)
        - [Start Function](#start-function)
        - [Exports](#exports)
        - [Imports](#imports)
        - [Modules](#modules-1)
- [__Execution - 执行__](#__execution---执行__)
    - [Conventions](#conventions-1)
        - [Prose Notation](#prose-notation-1)
        - [Formal Notation](#formal-notation-1)
    - [Runtime Structure](#runtime-structure)
    - [Values](#values)
        - [Results](#results)
        - [Store](#store)
        - [Addresses](#addresses)
        - [Module Instances](#module-instances)
        - [Function Instances](#function-instances)
        - [Table Instances](#table-instances)
        - [Memory Instances](#memory-instances)
        - [Global Instances](#global-instances)
        - [Export Instances](#export-instances)
        - [External Values](#external-values)
        - [Stack](#stack)
        - [Administrative Instructions](#administrative-instructions)
    - [Numerics](#numerics)
        - [Representations](#representations)
        - [Integer Operations](#integer-operations)
        - [Floating-Point Operations](#floating-point-operations)
        - [Conversions](#conversions)
    - [Instructions](#instructions-1)
        - [Numeric Instructions](#numeric-instructions-2)
        - [Parametric Instructions](#parametric-instructions-2)
        - [Variable Instructions](#variable-instructions-2)
        - [Memory Instructions](#memory-instructions-2)
        - [Control Instructions](#control-instructions-2)
        - [Blocks](#blocks)
        - [Function Calls](#function-calls)
        - [Expressions](#expressions-2)
    - [Modules](#modules-2)
        - [External Typing](#external-typing)
        - [Import Matching](#import-matching)
        - [Allocation](#allocation)
        - [Instantiation](#instantiation)
        - [Invocation](#invocation)
- [__Binary Format - 二进制格式__](#__binary-format---二进制格式__)
    - [_Conventions - 约定_](#_conventions---约定_)
        - [Grammar - 语法](#grammar---语法)
        - [Auxiliary Notation](#auxiliary-notation)
        - [Vectors](#vectors-1)
    - [Values](#values-1)
        - [Bytes](#bytes)
        - [Integers](#integers)
        - [Floating-Point](#floating-point)
        - [Names](#names-1)
    - [Types](#types-1)
        - [Value Types](#value-types)
        - [Result Types](#result-types)
        - [Function Types](#function-types-1)
        - [Limits](#limits-2)
        - [Memory Types](#memory-types-1)
        - [Table Types](#table-types-1)
        - [Global Types](#global-types-2)
    - [Instructions](#instructions-2)
        - [Control Instructions](#control-instructions-3)
        - [Parametric Instructions](#parametric-instructions-3)
        - [Variable Instructions](#variable-instructions-3)
        - [Memory Instructions](#memory-instructions-3)
        - [Numeric Instructions](#numeric-instructions-3)
        - [Expressions](#expressions-3)
    - [_Modules - 模块_](#_modules---模块_)
        - [Indices - 索引](#indices---索引-1)
        - [Sections - 段](#sections---段)
        - [Custom Section - 自定义 section](#custom-section---自定义-section)
        - [Type Section](#type-section)
        - [Import Section](#import-section)
        - [Function Section](#function-section)
        - [Table Section](#table-section)
        - [Memory Section](#memory-section)
        - [Global Section](#global-section)
        - [Export Section](#export-section)
        - [Start Section](#start-section)
        - [Element Section](#element-section)
        - [Code Section](#code-section)
        - [Data Section](#data-section)
        - [Modules](#modules-3)
- [__Text Format - 文本格式__](#__text-format---文本格式__)
    - [Conventions](#conventions-2)
        - [Grammar](#grammar)
        - [Abbreviations](#abbreviations)
        - [Contexts](#contexts-1)
        - [Vectors](#vectors-2)
    - [Lexical Format](#lexical-format)
        - [Characters](#characters)
        - [Tokens](#tokens)
        - [White Space](#white-space)
        - [Comments](#comments)
    - [Values](#values-2)
        - [Integers](#integers-1)
        - [Floating-Point](#floating-point-1)
        - [Strings](#strings)
        - [Names](#names-2)
        - [Identifiers](#identifiers)
    - [Types](#types-2)
        - [Value Types](#value-types-1)
        - [Result Types](#result-types-1)
        - [Function Types](#function-types-2)
        - [Limits](#limits-3)
        - [Memory Types](#memory-types-2)
        - [Table Types](#table-types-2)
        - [Global Types](#global-types-3)
    - [Instructions](#instructions-3)
        - [Labels](#labels)
        - [Control Instructions](#control-instructions-4)
        - [Parametric Instructions](#parametric-instructions-4)
        - [Variable Instructions](#variable-instructions-4)
        - [Memory Instructions](#memory-instructions-4)
        - [Numeric Instructions](#numeric-instructions-4)
        - [Folded Instructions](#folded-instructions)
        - [Expressions](#expressions-4)
    - [Modules](#modules-4)
        - [Indices](#indices)
        - [Types](#types-3)
        - [Type Uses](#type-uses)
        - [Imports](#imports-1)
        - [Functions](#functions-1)
        - [Tables](#tables-1)
        - [Memories](#memories-1)
        - [Globals](#globals-2)
        - [Exports](#exports-1)
        - [Start Function](#start-function-1)
        - [Element Segments](#element-segments-2)
        - [Data Segments](#data-segments-1)
        - [Modules](#modules-5)
- [__Appendix - 附录__](#__appendix---附录__)
    - [_Embedding - 嵌入_](#_embedding---嵌入_)
        - [Store](#store-1)
        - [Modules](#modules-6)
        - [Exports](#exports-2)
        - [Functions](#functions-2)
        - [Tables](#tables-2)
        - [Memories](#memories-2)
        - [Globals](#globals-3)
    - [Implementation Limitations](#implementation-limitations)
        - [Syntactic Limits](#syntactic-limits)
        - [Validation](#validation)
        - [Execution](#execution)
    - [Validation Algorithm](#validation-algorithm)
        - [Data Structures](#data-structures)
        - [Validation of Opcode Sequences](#validation-of-opcode-sequences)
    - [_Custom Sections - 自定义段_](#_custom-sections---自定义段_)
        - [Name Section](#name-section)
    - [Soundness](#soundness)
        - [Values and Results](#values-and-results)
        - [Store Validity](#store-validity)
        - [Configuration Validity](#configuration-validity)
        - [Administrative Instructions](#administrative-instructions-1)
        - [Store Extension](#store-extension)
        - [Theorems](#theorems)

<!-- /TOC -->

# __Introduction - 引言__

## _Introduction - 简介_

### Design Goals - 设计目标

### Scope

### Dependencies - 依赖

WebAssembly依赖两个现有标准：

* [IEEE 754-2008](http://ieeexplore.ieee.org/document/4610935/)
* [Unicode](http://www.unicode.org/versions/latest/)

>注意：
>

## _Overview - 概览_

### Concepts - 概念

### Semantic Phases - 语言阶段

# __Structure - 结构__

## Conventions - 约定

### Grammar Notation - 语法符号

### Auxiliary Notation - 辅助符号

### Vectors

## Values - 值

### Bytes - 字节

### Integers - 整数

### Floating-Point - 浮点数

### Names

## Types - 类型

### Value Types - 值类型

### Result Types - 结果类型

### Function Types - 函数类型

### Limits

### Memory Types - 内存类型

### Table Types - 表类型

### Global Types

### External Types

## Instructions - 指令

### Numeric Instructions

### Parametric Instructions

### Variable Instructions

### Memory Instructions

### Control Instructions

### Expressions

## Modules - 模块

### Indices - 索引

### Types - 类型

### Functions - 函数

### Tables - 表

### Memories - 内存

### Globals

### Element Segments

### Data Segments - 数据段

### Start Function - 开始函数

### Exports - 导出

### Imports - 导入

# __Validation - 验证__

## Conventions

### Contexts

### Prose Notation

### Formal Notation

## Types

### Limits

### Function Types

### Table Types

### Memory Types

### Global Types

## Instructions

### Numeric Instructions

### Parametric Instructions

### Variable Instructions

### Memory Instructions

### Control Instructions

### Instruction Sequences

### Expressions

## Modules

### Functions

### Tables

### Memories

### Globals

### Element Segments

### Data Segments

### Start Function

### Exports

### Imports

### Modules

# __Execution - 执行__

## Conventions

### Prose Notation

### Formal Notation

## Runtime Structure

## Values

### Results

### Store

### Addresses

### Module Instances

### Function Instances

### Table Instances

### Memory Instances

### Global Instances

### Export Instances

### External Values

### Stack

### Administrative Instructions

## Numerics

### Representations

### Integer Operations

### Floating-Point Operations

### Conversions

## Instructions

### Numeric Instructions

### Parametric Instructions

### Variable Instructions

### Memory Instructions

### Control Instructions

### Blocks

### Function Calls

### Expressions

## Modules

### External Typing

### Import Matching

### Allocation

### Instantiation

### Invocation

# __Binary Format - 二进制格式__

## _Conventions - 约定_

WebAssembly [模块](https://webassembly.github.io/spec/core/intro/overview.html#module)的二进制格式就是其[抽象语法](https://webassembly.github.io/spec/core/syntax/modules.html#syntax-module)的密集线性编码。[1]

此格式由 _attribute grammar_ 定义，whose only terminal symbols are [bytes](https://webassembly.github.io/spec/core/syntax/values.html#syntax-byte). A byte sequence is a well-formed encoding of a module if and only if it is generated by the grammar.

Each production of this grammar has exactly one synthesized attribute: the abstract syntax that the respective byte sequence encodes. 因此，attribute grammar 隐式的定义了解码函数。

Except for a few exceptions, the binary grammar closely mirrors the grammar of the abstract syntax.

> 注意：
>

WebAssembly 模块的二进制格式文件扩展名推荐使用`.wasm`，多媒体类型推荐使用`application/wasm`。

> [1] Additional encoding layers – for example, introducing compression – may be defined on top of the basic representation defined here. However, such layers are outside the scope of the current specification.

### Grammar - 语法

以下约定用于定义二进制格式的语法规则。它们反映了用于[抽象语法](https://webassembly.github.io/spec/core/syntax/conventions.html#grammar)的约定。为了区分二进制语法的符号和抽象语法的符号，__typewriter__ font is adopted for the former.

### Auxiliary Notation

### Vectors

## Values

### Bytes

### Integers

### Floating-Point

### Names

## Types

### Value Types

### Result Types

### Function Types

### Limits

### Memory Types

### Table Types

### Global Types

## Instructions

### Control Instructions

### Parametric Instructions

### Variable Instructions

### Memory Instructions

### Numeric Instructions

### Expressions

## _Modules - 模块_

模块的二进制编码分为不同的 _section_ 。大部分section与模块的组件一对一对应，而[函数定义](https://webassembly.github.io/spec/core/syntax/modules.html#syntax-func)被分成两个section：函数声明位于[function section](https://webassembly.github.io/spec/core/binary/modules.html#binary-funcsec)，函数体位于[code section](https://webassembly.github.io/spec/core/binary/modules.html#binary-codesec)。

> 注意
> This separation enables parallel and streaming compilation of the functions in a module.

### Indices - 索引

### Sections - 段

一个段由三部分组成：

* 一个字节的 section id
* [u32](https://webassembly.github.io/spec/core/syntax/values.html#syntax-int)个字节的内容
* 实际内容，其具体结构由 section id 决定

所有 section 都是可选的。被省略的 section 与空内容的 section 是等价的。

> 注意
>

以下是 section id 的定义：

Id | Section | 
------------ | ------------- | 
0 | [custom section](https://webassembly.github.io/spec/core/binary/modules.html#binary-customsec)  | 
1 | [type section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
2 | [import section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
3 | [function section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
4 | [table section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
5 | [memory section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
6 | [global section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
7 | [export section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
8 | [start section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
9 | [element section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
10 | [code section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 
11 | [data section](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec)  | 

### Custom Section - 自定义 section



### Type Section

### Import Section

### Function Section

### Table Section

### Memory Section

### Global Section

### Export Section

### Start Section

### Element Section

### Code Section

### Data Section

### Modules

# __Text Format - 文本格式__

## Conventions

### Grammar

### Abbreviations

### Contexts

### Vectors

## Lexical Format

### Characters

### Tokens

### White Space

### Comments

## Values

### Integers

### Floating-Point

### Strings

### Names

### Identifiers

## Types

### Value Types

### Result Types

### Function Types

### Limits

### Memory Types

### Table Types

### Global Types

## Instructions

### Labels

### Control Instructions

### Parametric Instructions

### Variable Instructions

### Memory Instructions

### Numeric Instructions

### Folded Instructions

### Expressions

## Modules

### Indices

### Types

### Type Uses

### Imports

### Functions

### Tables

### Memories

### Globals

### Exports

### Start Function

### Element Segments

### Data Segments

### Modules

# __Appendix - 附录__

## _Embedding - 嵌入_

### Store

### Modules

### Exports

### Functions

### Tables

### Memories

### Globals

## Implementation Limitations

### Syntactic Limits

### Validation

### Execution

## Validation Algorithm

### Data Structures

### Validation of Opcode Sequences

## _Custom Sections - 自定义段_

### Name Section

## Soundness

### Values and Results

### Store Validity

### Configuration Validity

### Administrative Instructions

### Store Extension

### Theorems
