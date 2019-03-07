---
title: 解决ElementUI中pagination组件页码问题
date: 2018-09-17 19:39:18
tags:
  - ElementUI
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/IMG_6183_w.JPG
---

# 环境
- element-ui 2.4.5

# 起因

在使用 Element UI 中的 pagination 时出现了一个问题。点击页页码可以正常跳转，跳转到对应页面后，再跳转到其他路由，然后返回。页码又被重置为 1 。

奇怪的是，我已经将页码数据存储到 vuex 中，而且通过 vue devtool 查看确实页码数据并不为 1 。

那么，究竟是哪里出问题了呢？

# 猜测

上网查了很多博客以及官方的 issue 。其中 [这篇](https://lhajh.github.io/ui/2017/09/15/elementui-problem-collection.html) 博客中的观点给我了一点启发。

> 删除 jumper 中的数字后，当前页跳到第 1 页并不是点击 next 直接导致的，而是 jumper 失焦后其值此时为空字符串，应用值到 internalCurrentPage 前会校验设置的页码，部分源码如下，会被设置为 1

根据这个线索我在考虑，是不是因为由于我获取数据使用的 axios ，请求是异步的，在组件初始化之后，pagination 的子组件 jumper 并没有获取到 page 。

# 解决方法

触发条件：

1. total 初始化为 0（number）
2. 同步操作中改变 currentPage 的值
3. 异步操作获取 total 的值
4. 分页组件拿到不合法的 currentPage 会将页码重置为 1，即使之后修改了 total ，也不会重置 currentPage

解决方法：将 total 初始值置为 null 而不是 0



# 本文参考
- 封面摄自千岛湖
- [ElementUI 问题集合](https://lhajh.github.io/ui/2017/09/15/elementui-problem-collection.html)
- [Element issues](https://github.com/ElemeFE/element/issues)