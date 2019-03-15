---
title: My Name is AGAL, and I Come from Adobe – Part1
layout: post
---

想像有一天人类灭绝。

想像在人类灭绝后，附着时间流逝，所有纸质文档变得破烂而无法阅读。想像之后有外星生物发现一个由人类制造的DVD或固态硬盘，其中包含了某些信息。

外星生物该如何破译这些信息。他们如何才能找到理解这些“人造物品”中包含的信息的关键？

更一般地说，如果他们接触到我们的技术产物，如何才能真正地理解我们的发现与我们的技术，他们是否需要这些技术才能理解技术本身。

Adobe 最近引进了一种被称为AGAL (Adobe Graphics Assembly Language)的新语言。它是Molehill项目的一部分，其目的是生成所谓的"着色器(shaders)"：决定如何在场景中渲染3D模型的短小程序。

Shaders非常酷。它们生成让人惊叹的渲染效果。它们也比普通的AS代码难写。

AGAL形如：

```
//vertex shader
m44 op, va0, vc0 // pos to clipspace
mov v0, va1 // copy uv

//pixel shader
tex ft1, v0, fs0 <2d,linear,nomip>
mov oc, ft1
```

是不是跟天书一样？理解它的关键是什么？

现在阶段的AGAL技术文档太少了。让我们揭开这神秘语言的面纱。

## **操作码(The OpCode)**

着色器的每一行代码都是一个命令，由3个字符组成所谓的"操作码(opcode)"。

AGAL的代码行有着如下语法：

```
<opcode> <destination> <source 1> <source 2 or sampler>
```

这是关键。牢记这个语法AGAL将不再是不可阅读的字符堆。

根据实际命令，紧跟操作码的可以是一个目标操作数和一或两个源操作数。目标操作数和源操作数都是"寄存器(registers)"：供着色器使用的在GPU中的细小内存区域。

AGAL有着30多个不同特性的操作码。完整的操作码清单可以在Molehill的文档中找到。这里列出了最常用的部分操作码。

* mov  将数据从源寄存器1移动到目标寄存器，component-wise
* add  目标寄存器=源寄存器1 + 源寄存器2，component-wise
* sub  目标寄存器=源寄存器1 - 源寄存器2，component-wise
* mul  目标寄存器=源寄存器1 * 源寄存器2，component-wise
* div  目标寄存器=源寄存器1 / 源寄存器2，component-wise
* dp3  源寄存器1 点乘 源寄存器2（3 components）
* dp4  源寄存器1 点乘 源寄存器2（4 components）
* m44  4维向量（源寄存器1） 右乘 4X4矩阵（源寄存器2）
* tex  纹理取样。源寄存器1存储纹理坐标，源寄存器2存储纹理

## **AGAL寄存器**

寄存器是供AGAL程序(着色器)运行期间使用的GPU内存区域。寄存器可用于AGAL命令的源操作数(源寄存器)和目标操作数(目标寄存器)。

你可以通过这些寄存器向着色器传入参数。

每个寄存器有128位宽，意味着包含4个浮点数。每一个浮点数被称为寄存器的"组件(component)"。寄存器组件可以通过"坐标选择器(xyzw)"或"颜色选择器(rgba)"访问。

所以，一个寄存器的第一个组件可以有两种访问方式：

```
<register name>.x 或 <register name>.r
```

上面的一些操作码如"add"，按组件(component-wise)进行操作。这意味着这些操作是各寄存器对应的组件之间的操作。

有6种类型的寄存器

### **属性寄存器（Attribute Registers）**

这些寄存器指向由顶点着色器(vertex shader)输入的VertexBuffer(顶点缓存器)。因此它们只用于顶点着色器。

通过`Context3D::setVertexBufferAt(index)`函数指派一个VertexBuffer给相应的属性寄存器。

最终着色器通过属性寄存器的索引号访问对应的寄存器，语法如：va\<n>，其中\<n>即为索引号，总计8个属性寄存器

### **常量寄存器(Constant Registers)**

这些寄存器用于从AS代码中向着色器传递参数。这个操作通过`Context3D::setProgramConstants()`函数族完成。

顶点着色器使用语法：vc\<n>，其中\<n>即为索引号，有总计128个常量寄存器

片段(像素)着色器使用语法：fc\<n>，其中\<n>即为索引号，有总计28个常量寄存器

### **临时寄存器(Temporary Registers)**

这些寄存器用于着色器的临时运算产生的中间结果

顶点着色器使用语法：vt\<n>，其中\<n>即为索引号，有总计8个临时寄存器

片段(像素)着色器使用语法：ft\<n>，其中\<n>即为索引号，有总计8个临时寄存器

### **输出寄存器(Output Registers)**

这些寄存器用于存储顶点着色器和片段(像素)着色器各自的运算结果。对于顶点着色器而言，运算结果为顶点在裁剪空间的位置。对于片段(像素)着色器而言则是像素颜色。

顶点着色器访问语法：op

片段(像素)着色器使用语法：oc

两种输出寄存器均只有一个。

### **变量寄存器(Varying Registers)**

这些寄存器用于从顶点着色器向片段(像素)着色器传递数据。这些数据都经过GPU的适当插值，所以片段(像素)着色器总是收到用于处理当前像素的正确数据。

一般通过这种方式传递的数据为顶点颜色或纹理的UV坐标。

使用语法：v\<n>，其中\<n>即为索引号，总计有8个变量寄存器

### **纹理取样器(Texture Samplers)**

纹理取样器寄存器用于通过UV坐标从纹理中提取颜色值。

纹理通过`Centext3D::setTextureAt()`函数指定。

纹理取样器寄存器使用语法：fs\<n> \<flags>，其中\<n>即为取样器寄存器索引号，而\<flags>则为一或多个用于指定取样方式的标志。

\<flags>是用逗号隔开的字符串集合，定义如下：

* 纹理维数(texture dimension)，可选值：2d，3d，cube
* 贴图处理(mip mapping)，可选值：nomip，mipnone，mipnearest, mipliner
* 纹理过滤(textrue filtering)，可选值：nearest，linear
* 纹理重复(textrue repeat)，可选值：repeat，wrap，clamp

所以，一个不需要贴图处理并且是线性过滤的标准2D纹理，可以用如下命令取样到临时寄存器ft1：

```
tex ft1, v0, fs0 <2d,linear,nomip>
```

变量寄存器v0保存着经过适当插值的UV坐标。

## **顶点着色器和片段(像素)着色器示例**

现在让我们回到着色器例子并解释其操作。

假设我们存于VertexBuffer的顶点从偏移0开始是顶点位置，从偏移3开始是UV坐标。

我们希望顶点着色器把顶点位置转换到裁剪空间并把UV坐标传递给片段(像素)着色器。

执行代码如下:

```
m44 op, va0, vc0 // pos to clipspace
mov v0, va1 // copy uv
```

第一行执行了一个4X4的矩阵乘法：输入数据va0，转换矩阵vc0，其结果是把顶点从模型空间转移到裁剪空间。
转换矩阵可以通过如下AS代码传递给着色器：

```
Context3D::setProgramConstantsFromMatrix(Context3DProgramType.VERTEX, 0, matrix, true)
```

第二行，顶点着色器把UV坐标复制到变量寄存器v0以便进行适当的插值并传递给片段(像素)着色器。
片段(像素)着色器进行纹理取样，并输出颜色。

```
tex ft1, v0, fs0 <2d, linear, nomip>
mov oc, ft1
```

所以，顶点着色器和片段(像素)着色器的职责是把3d模型转换到3d屏幕，并进行纹理映射。
待续。。。