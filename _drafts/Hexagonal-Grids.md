---
layout: post
---

<!-- TOC -->

- [[Hexagonal Grids][Hexagonal Grids] - 六边形网格](#hexagonal-gridshexagonal-grids---六边形网格)
    - [Geometry - 几何定义](#geometry---几何定义)
        - [Angles - 角度](#angles---角度)
        - [Size and Spacing - 大小和间距](#size-and-spacing---大小和间距)
    - [Coordinate Systems - 坐标系统](#coordinate-systems---坐标系统)
        - [Offset coordinates - 偏移坐标法](#offset-coordinates---偏移坐标法)

<!-- /TOC -->

# [Hexagonal Grids][Hexagonal Grids] - 六边形网格

六边形网格用于某些游戏，但不如四边形网格那么直观和常见。过去20多年，我收集了不少六边形网格的[资料][collecting hex grid resources]，它们大部分都基于[Charles Fu][Charles Fu]和[Clark Verbrugge][Clark Verbrugge]的指南，最终在这份指南里我以最简洁的代码写出了最优雅的实现。我将会描述制作六边形网格的各种方法和它们之间的联系以及一些常见的算法。本指南有不少互动内容。

为了便于阅读和理解，本指南的示例代码以伪代码编写。[实现指南][The implementation guide]有C++, Javascript, C#, Python, Java, Typescript等语言的实现代码。

## Geometry - 几何定义

六边形是有6条边的多边形。正六边形的各条边长度相等。我们假定本指南所提到的所有六边形都是正六边形。垂直列（平顶）和水平行（尖顶）是六边形网格的两种经典方向。

六边形有6条边和6个角。（在六边形网格中，六边形的）每条边由2个六边形共享，第个角由3个六边形共享。更多关于中心点、边和角的资料，可以参考我的另一篇关于[网格][my article on grid parts]的文章，里面分析了3角形、4边形和6边形。

### Angles - 角度

正六边形的（每个）内角是$120^\circ$。（正六边形）可以分成6个楔形，每个楔形都是一个等边三角形，其（每个）内角是$60^\circ$。每个角距离中心点 _size_ 单位长度。计算代码如下

```
#尖顶
function pointy_hex_corner(center, size, i):
    var angle_deg = 60 * i - 30°
    var angle_rad = PI / 180 * angle_deg
    return Point(center.x + size * cos(angle_rad),
                 center.y + size * sin(angle_rad))

#平顶
function flat_hex_corner(center, size, i):
    var angle_deg = 60 * i
    var angle_rad = PI / 180 * angle_deg
    return Point(center.x + size * cos(angle_rad),
                 center.y + size * sin(angle_rad))
```

填充六边形需要收集`hex_corner(..., 0)`到`hex_corner(..., 5)`这6个多边形顶点。而绘制六边形的轮廓只要顺序连接前面6个顶点即可。

两种方向（的六边形网格）的区别在于旋转和由此产生的角度差异：平顶六边形分别是$0^\circ$, $60^\circ$, $120^\circ$, $180^\circ$, $240^\circ$, $300^\circ$，而尖顶六边形分别是$30^\circ$, $90^\circ$, $150^\circ$, $210^\circ$, $270^\circ$, $330^\circ$。要注意本指南的坐标系Y轴正方向指向下（角度顺时针增加），如果你使用的坐标系Y轴正方向指向上（角度逆时针增加），那你需要做些调整。

### Size and Spacing - 大小和间距

接下来我们要把多个六边形拼到一起。

对于平顶方向（六边形），六边形的宽度为$w=2*size$，高度为$h=\sqrt{3}*size$，$\sqrt3$来源于$\sin(60^\circ)$。相邻的两个六边形的中心点水平相距$w * 3/4$，垂直相距$h$

对于尖顶方向（六边形），六边形的宽度为$w=\sqrt{3}*size$，高度为$h=2*size$，$\sqrt3$来源于$\sin(60^\circ)$。相邻的两个六边形的中心点水平相距$w$，垂直相距$h * 3/4$

Some games use pixel art for hexagons that does not match an exactly regular polygon. The angles and spacing formulas I describe in this section won’t match the sizes of your hexagons. The rest of the article, describing algorithms on hex grids, will work even if your hexagons are stretched or shrunk a bit, and I explain on the implementation page how to handle stretching.

## Coordinate Systems - 坐标系统

现在我们需要把很多六边形拼成（一个完整的六边形）网格。对于四边形网格，明显只有一种拼法。而六边形网格则有多种方法。I like cube coordinates for algorithms and axial or doubled for storage.

### Offset coordinates - 偏移坐标法

















[Hexagonal Grids]: https://www.redblobgames.com/grids/hexagons/
[collecting hex grid resources]: http://www-cs-students.stanford.edu/~amitp/gameprog.html#hex
[Charles Fu]: http://www-cs-students.stanford.edu/~amitp/Articles/Hexagon2.html
[Clark Verbrugge]: http://www-cs-students.stanford.edu/~amitp/Articles/HexLOS.html
[The implementation guide]: https://www.redblobgames.com/grids/hexagons/implementation.html
[my article on grid parts]: http://www-cs-students.stanford.edu/~amitp/game-programming/grids/