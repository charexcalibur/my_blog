---
title: 详解如何使用CSS实现守望先锋的loading动画效果
date: 2017-02-02 20:00:31
tags:
- CSS动画
categories:
- 技术
---



首先先来介绍这个动画整体的构造，最内层是 OW 的图标

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/overwatch.png)

外层有五个圆环组成。HTML结构如下：

```html

              <div class="owloading">
                            <div class="overwatch">
                                <div class="across-line"></div>
                                <div class="taper"></div>
                            </div>
                            <div class="inner-ring3">
                                <div class="inner-ring3-p1-c">
                                    <div class="p1-an">
                                        <div class="inner-ring3-p1-r"></div>
                                    </div>
                                </div>
                                <div class="inner-ring3-p2-c">
                                    <div class="p2-an">
                                        <div class="inner-ring3-p2-r"></div>
                                    </div>
                                </div>
                            </div>
                            <div class="inner-ring"></div>
                            <div class="inner-ring2">
                                <div class="inner-ring2-c">
                                    <div class="inner-ring2-r"></div>
                                </div>
                            </div>
                            <div class="outer-ring"></div>
                            <div class="outer-ring2"></div>
                        </div>
                    </div>
```

我们来一层一层解读。首先是 OW 图标，`<div class="overwatch"></div>`用来实现大圆环

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/bigcircle.png)

其下有两个子`div`，`<div class="across-line"></div>`用来实现圆环上方的割裂效果

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/across-line.png)

`<div class="taper"></div>`用来实现如图所示的效果

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/taper.png)

那么问题来了。。。

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/w.png)

这块区域是哪里冒出来的呢？我打开chrome暗中观察了一番，原来这两部分是通过父级`div`的伪元素来实现的。

参考下图：

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/bcbefore.png)

分析完了页面结构，我们再来看样式。也先从`<div class="overwatch"></div>`开始。

```css

    .overwatch {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 160px;
        height: 160px;
        margin: -80px 0px 0px -80px;
        background-color: transparent;
        border-radius: 50%;
        border: 20px solid #B6B8C0;
    }
 ```

很简单，绝对定位，通过 `border-radius: 50%;和border` 实现圆环。接着来看右边这个伪元素 before (左边同理)。

```css

    .overwatch:before {
        content: "";
        height: 20px;
        width: 66px;
        background: #B6B8C0;
        position: absolute;
        bottom: 44px;
        right: -9px;
        transform: rotate(45deg) skew(45deg);
        transform-origin: bottom left;
    }
 ```

绝对定位，否则无法呈现图像，设置宽高和颜色，通过 `transform: rotate(45deg) skew(45deg);` 对这个矩形进行旋转和倾斜。通过 `transform-origin: bottom left;` 来改变位置，这里我试了下，其实用 `transform-origin: left bottom;` 也没差，效果一样。可能是因为 x 轴值没有 bottom ，所以当 bottom 出现在x轴位置时，浏览器默认把它移到y轴去了。具体的  tranform-origin 的属性效果，可以看这个[在线演示](http://www.w3school.com.cn/example/css3/demo_css3_transform-origin_3D.html)。

再来看上面提到过的割裂效果的样式是如何写的。也是左右各一个伪元素，我们来看右边这个。

```css

    .overwatch div.across-line:before {
        content: "";
        height: 4px;
        width: 22px;
        background: #282828;
        position: absolute;
        top: 15px;
        right: -5px;
        transform: rotate(-45deg);
        transform-origin: bottom left;
        z-index: 10;
    }
 ```

从代码来看与前面并没有什么差别，设置宽高之后进行旋转和定位。要注意一点，需要将此元素的堆叠顺序也就是 `z-index` 属性高于圆环，才能实现割裂效果。

`<div class="taper"></div>` 的实现方式类似，此处就不详细描述了。

OW 图标部分解释完毕，其实构图的精髓在于如何 **活用伪元素:after和:after，以及使用定位来构造图形**。

接着来看外圈部分，外圈部分的呈现除了使用了伪元素之外，还通过对伪元素使用 `clip和animation属性` 来构成运动的圆环。详细分析一波`<div class="inner-ring3"></div>`。

老样子先来看 HTML 结构。DOM 树大概是这么个情况：

                       <div class="inner-ring3">
                       ____________|____________
                      |                         |
    <div class="inner-ring3-p1-c">    <div class="inner-ring3-p2-c">
                      |                         |
              <div class="p1-an">      <div class="p2-an">
                      |                         |
    <div class="inner-ring3-p1-r">    <div class="inner-ring3-p2-r">

`.innner-ring3` 的样式如下：

```css

       .inner-ring3 {
           position: absolute;
           top: 50%;
           left: 50%;
           width: 200px;
           height: 200px;
           margin: -100px 0px 0px -100px;
           background-color: transparent;
           border-radius: 50%;
           animation: ring6 2s infinite linear;
       }
       @keyframes ring6 {
           0% {
               transform: rotate(0deg);
           }
           100% {
               transform: rotate(360deg);
           }
       }
 ```

是一个透明圆旋转的圆。

`<div class="inner-ring3-p1-c">` 样式为：

```css

    .inner-ring3 .inner-ring3-p1-c {
          position: relative;
          float: left;
          width: 100px;
          height: 200px;
          overflow: hidden;
          }
```

这里可以看出这个 `div` 的高和其父级一样，宽只有父级的一半。这里的 `overflow: hidden;` 其实是为了后面的铺垫。当父级旋转时，此 `div` 也会同时旋转。

`<div class="p1-an">` 样式为：

```css

    .inner-ring3 .p1-an {
        width: 200px;
        height: 200px;
        position: absolute;
        top: 0;
        left: 0;
        animation: ring5 2s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    }
    @keyframes ring5 {
        0% {
            transform: rotate(0deg);
        }
        64.5% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(180deg);
        }
    }
 ```

此时这块 200px 的正方形也会开始旋转，keyframes ring5和 keyframe ring6 有一个时间差。具体会通过下面这个 div 来实现。

`<div class="inner-ring3-p1-r">` 样式为：

```css

    .inner-ring3 .inner-ring3-p1-r {
        width: 200px;
        height: 200px;
        border: 10px solid transparent;
        border-radius: 50%;
        position: absolute;
        top: 0;
        left: 0;
        border-top: 10px solid rgba(189, 186, 62, 0.25);
        border-left: 10px solid rgba(189, 186, 62, 0.25);
        transform: rotate(-45deg);
        animation: ring4 2s linear infinite;
    }
    @keyframes ring4 {
        0% {
            transform: rotate(-45deg);
        }
        35.5% {
            transform: rotate(-45deg);
        }
        50% {
            transform: rotate(135deg);
        }
        100% {
            transform: rotate(135deg);
        }
    }
 ```

这是使用 `border-top和border-left` 来获得半圆环得到的效果如图

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/ow/-45deg.png)

此时设置 ```transform: rotate(-45deg);``` 来使半圆环位置在 ```<div class="inner-ring3-p1-c">``` 内，配合该元素的```overflow: hidden;``` ,当圆环转出此矩形区域是将会消失。这样就能达到动画中所示的效果了。

后面的几个圆环效果基本与之类似。无非是使用了 ```clip: rect()``` 来将圆环裁成不同长短的块状。

# 本文参考

- 《[前端网：用纯 css3 做了一个在守望先锋加载地图时的 loading](http://www.qdfuns.com/notes/28003/56008504355cf20cfaa08dcc022b6e25.html)》
- [w3school](http://www.w3school.com.cn/h.asp)