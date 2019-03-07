---
title: 浅谈before和after伪元素
date: 2017-02-01 21:58:24
tags:
- CSS
- 伪元素
- CSS动画
categories:
- 技术
---
# ::before和::after伪元素

前一段时间在玩 CSS 动画，真好玩啊。收集在了[Animate](https://charexcalibur.github.io/animate/)上，很有意思，很炫酷。

读了源码，发现代码里大量使用了 ::before 和 ::after 伪元素。之前看 CSS 文档的时候，记得有 CSS 伪类，那么伪元素和伪类有什么关系呢？<!--more-->

**CSS3中为了区别伪元素和伪类元素而使用了双冒号**。所以在使用CSS3动画时建议使用新版本的浏览器。

```css
    .outer::before {
        content: "";
        width: 200px;
        height: 200px;
        display: block;
        position: absolute;
        border: 2px solid rgba(161, 164, 176, 0.7);
        border-radius: 50%;
        top: -2px;
        left: -2px;
        box-sizing: border-box;
        clip: rect(0px, 21px, 54px, 0px);
        animation: rotate infinite;
        animation-duration: 4s;
        animation-timing-function: ease-in-out;
    }
```

看这段 CSS 可以发现 ::before 伪元素下有一个特有的属性content 。
在 w3school 上对 content 属性是这样说明的：
>content 属性用来生成内容。在默认情况下是行内内容，不过该内容常见的类型可以用属性 display 来控制。

之前有提到过，当行内元素的 display 属性设置成 block 来实现块级化。所以在上代码中使用了 display:block; 。

同时，在这段代码中也使用了 box-sizing 属性，当box-sizing: border-box; 时，此元素的内边距和边框将不会增加它的宽度。

# 基本用法

前文中提到，两个伪类下的 content 属性，用于在 CSS 渲染中向元素的逻辑上的头部或尾部添加内容，**这些添加不会改变文档内容，也不会出现在DOM中，不可复制，仅仅是在CSS渲染层中加入**。在[张鑫旭的这篇博客](http://www.zhangxinxu.com/wordpress/2015/04/before-after-pseudo-elements-special-features/)对伪元素算在文档内容里有实例解答。常用的有以下几个值：

|值    |说明|
|----|----|
|String|设置Content到指定的文本|
|attr()|调用当前的属性，可以方便的将图片的alt或者链接的href地址显示出来|
|url()|引用媒体文件|
|counter()|调用计数器，可以不使用列表元素实现序号功能。|

更多用法大家可以自己去写一下。或查阅 [runoob](http://www.runoob.com/cssref/pr-gen-content.html)上的文档。

# 在动画特效中的使用

在动画特效中，我们需要将 content 设置成 ""，然后配合position: absolute; 和 display: block; 就能是 after 和 before 形成两个容器，通过对这两个容器的操作，就可以实现动画和特效效果。

例如图中这两个部分圆环。

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/%E4%BC%AA%E5%85%83%E7%B4%A0.png)

HTML部分：

```html
    <div class="outer-ring"></div>
```

CSS部分：

```css
    .outer-ring {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 240px;
        height: 240px;
        margin: -120px 0px 0px -120px;
        background-color: transparent;
        border-radius: 50%;
        border: 6px solid transparent;
    }

    .outer-ring:before {
        content: "";
        display: block;
        position: absolute;
        top: -6px;
        left: -6px;
        width: 240px;
        height: 240px;
        border: 6px solid rgba(161, 164, 176, 0.5);
        border-radius: 50%;
        clip: rect(100px, 241px, 127px, 207px);
        animation: rotate3603 3s cubic-bezier(0.34, 0.07, 0.68, 0.93) infinite;
    }
    .outer-ring:after {
        content: "";
        display: block;
        position: absolute;
        top: -6px;
        left: -6px;
        width: 240px;
        height: 240px;
        border: 6px solid rgba(161, 164, 176, 0.5);
        border-radius: 50%;
        clip: rect(104px, 240px, 114px, 100px);
        animation: rotate3602 3s linear infinite reverse;
    }
```

从以上代码中我们可以看出，div 本身被设置成了透明属性的圆环，然后通过在 div 前后添加伪元素，实现两个圆环效果。分别使用 clip 属性将这两个圆环裁剪成不同的长短，同时添加不同的动画效果。最终就形成了两个围绕同一圆心旋转的部分圆环。

# 本文参考

- [张鑫旭博客](http://www.zhangxinxu.com/wordpress/2010/09/after%E4%BC%AA%E7%B1%BBcontent%E5%86%85%E5%AE%B9%E7%94%9F%E6%88%90%E5%B8%B8%E8%A7%81%E5%BA%94%E7%94%A8%E4%B8%BE%E4%BE%8B/)

- [你说不知道的CSS伪元素](http://justcoding.iteye.com/blog/2032627)

- [w3school](http://www.w3school.com.cn/)
