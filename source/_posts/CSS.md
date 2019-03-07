---
title: 几个常见的CSS核心知识
date: 2017-01-31 01:27:57
tags: CSS基础概念
categories:
- 技术
---
最近在研究CSS3的动画，借此机会，回顾一下CSS的基础核心知识。特地整理成此文，方便以后查阅。<!--more-->

# 元素类型

HTML的元素分为两种：

- 块级元素（block level element）
- 行内元素(inline element)

两者之间有以下几点区别：

- 块级元素会独占一行，也就是说，该元素无法与其他元素显示在同一行内，除非修改元素的display属性。而行内元素则会在同一行显示。
- 块级元素可以设置width,height属性，行内元素无法设置。
- 块级元素的width默认为100%，而行内元素则是根据其自身的内容或子元素来决定其宽度。

```<div>```是最常见的块级元素，```<span><a><img>```为常见的行内元素。

记得之前看过一篇博客是这样来描述的，


    .example {
        width: 100px;
        heigth: 100px;
    }


将一个```<div>```设置成上面的样式，是有效果的，因为它是块级元素，如果对```<span>```设置以上样式则没有效果。网上查了一下，有解决方案！**可以通过将```<span>```的display属性设置成block来实现行内元素的块级化**。

如果想要让元素在行内显示的同时，设置宽和高，可以这样写：

    display: inline-block;


**nline-block可以让元素对外呈行内元素，可以和其他元素出现在同一行；对内则让元素呈块级元素，可以改变其宽高**。

HTML的代码是顺序执行的，无CSS样式的HTML代码最终呈现的页面是根据元素出现的顺序和类型来排列的。**块级元素从上到下排列，行内元素从左到右排列**。这种无样式的情况下，元素的分布叫做**普通流**，浏览器会呈现元素的原始结构，同时所有元素会在页面上占据一定空间，空间的大小由其**盒模型**来决定。

# 盒模型

页面上显示的元素可以被看成是一个盒子，即盒模型（box model）。

![盒模型](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/box-model.png)
从图中可以看很出盒模型是由4部分组成的。分别为


    content -> padding -> border -> margin


一般来讲，一个元素的宽度(高度同理)，是这样来计算的：


    总宽度 = margin-left +　border-left + padding-left + width + padding-right + border-right + margin-right


符合W3C标准的浏览器认为一个元素的宽度只等于其content的宽度，其余都算是额外的。于是我们定义一个元素：


    .example {
        width: 200px
        height: 10px
        border: 5px;
        margin: 20px
    }


则它的最终宽度应该为：


    宽度 = width(200px) + padding(10px * 2) +　border(5px * 2) + margin(20px * 2) = 270px;


而在IE9及以下，最终宽度为：


    宽度 = width(200px) + margin(20px * 2) = 240px;


为了解决这个问题，W3C在CSS3中添加了Box-sizing这个属性。当我们设置box-sizing: border-box;时，border和padding就会被包含在宽高之内，和IE之前的标准是一样的。所以如果你的样式需要兼容低版本的IE，最好在CSS文件里加上以下代码：


    *,*:before,*:after {
        -moz-box-sizing: border-box;
        -webkit-box-sizing: border-box;
        box-sizing: border-box;
    }

此处还有两特殊情况：

- 无宽度 —— 绝对定位（position: absolute;）元素
- 无宽度 —— 浮动（float）元素

它们在页面上的均不占据空间（脱离普通流，浮在页面上，移动它们不会影响其他元素的定位）。这就涉及到了position和float这两个核心概念。

# position

position这个属性决定了元素如何定位。它的值大概有以下几种：

|position值      | 如何定位|
|---------     |-----   |
|static           |position的默认值。元素将定位到它的正常位置上。<br>其实也就是相当于没有定位。元素在页面上占据位置。<br>不能使用top right bottom left 移动元素。|
|relative       |相对定位，相对于元素的正常位置来进行定位。<br>元素在页面占据位置。可以使用top right bottom left 移动元素|
|absolute     |绝对定位，相对于最近一级（不是static的）父元素来进行定位。<br>元素在页面上不占据位置。<br>可以使用top right bottom left 移动元素|
|fixed       |绝对定位，相对于浏览器窗口来进行定位。<br>其余和absolute一样|
|inherit    | 从父元素继承position属性的值|

具体效果可以参考w3school的实例。

每个网页都可以看成是由一层一层页面堆叠起来的，可以看下图来理解。


![xyz](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/xyz.png)

position设置为relative时，元素依然在普通流中，位置是正常位置，可以通过left right来移动元素。会影响到其他元素的位置。

而当一个元素的position值为absolute或fixed的时候，会发生三件事：

1. 把该元素往Z轴方向移了一层，元素脱离了普通流，所以不会再占据原来的那层空间，还会覆盖下层的元素。
2. 该元素将变为块级元素，相当于给该元素设置了display:block;(给一个行内元素，如```<span>```,设置成absolute之后会发现它可以设置宽高了)。
3. 如果该元素是块级元素，元素的宽度由原来的width:100%(占据一行)，变为了auto。

由此可以看出，当position设置为absolute或fixed时，就没必要设置display为block了。


# float

float，就是元素浮动，有四个取值: left right none inherit。

关于float的几个要点：

1. 只有左右浮动，没有上下浮动
2. 元素设置float之后，它就会脱离普通流（和position:absolute;一样）,不在占据原来的那层空间，还会覆盖下一层的元素。
3. 浮动不会对该元素的上一个兄弟元素有任何影响。
4. 浮动之后，该元素的下一个兄弟元素会紧贴到该元素之前没有设置float的元素之后，也就是说该元素脱离了文档流，后一个元素补上位置。
5. 如果该元素的下一个兄弟元素中有行内元素（比如说文字），则会围绕该元素显示，类似**文字围绕图片**效果。
6. 下一个兄弟元素如果也设置了同一方向的float，则会紧挨着该元素之后显示。
7. 该元素将变为块级元素，相当于给该元素设置了display:block;(和position:absolute;一样)

以上是几个常见的CSS概念。


本文参考了：
- [w3school](http://www.w3school.com.cn/)
- [前端大全公众号](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551696&idx=1&sn=2627db03203b893ccb279b67c365cfb1&chksm=8025a011b7522907b2c3a59f71c77e277aad1ad5c108d9c389fe4d78903077043e8a3f00d1a2&scene=0#rd)


