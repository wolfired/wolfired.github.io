---
layout: post
---

[TOC]

## [特殊目录与脚本编译顺序][ScriptCompileOrderFolders]
一般情况下，你可以为U3D项目内的目录取任意名字，但U3D保留了一些名字用于表示一个目录具有特殊功能。其中有些目录会影响到脚本的编译顺序。本质上，有4个独立的脚本编译阶段，而一个脚本在哪个阶段编译由其父目录决定。

这解决了脚本的外部依赖问题。其基本原则是：不能引用在下一阶段才编译的任务东西。当前或之前阶段的所有东西都是可引用的。

举例来说：某种语言写的一个脚本引用了另外一种语言定义的一个类（如：一个UnityScript文件声明了一个变量，其类型由C#脚本定义），上面的原则在这里就是被引用的类必须在之前的阶段编译。

各编译阶段如下：




另外，**Assets**目录下的顶级目录**WebPlayerTemplates**里的脚本不会被编译。


[ScriptCompileOrderFolders]: https://docs.unity3d.com/550/Documentation/Manual/ScriptCompileOrderFolders.html