---
title: >-
  解决hexo迁移至mac后，报错Error: Cannot find module
  './build/Release/DTraceProviderBindings'的问题
date: 2018-08-12 22:44:33
tags:
  - hexo
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/IMG_6110.JPG
---

# 起因

换了 Mac 之后就想把之前在 Win 下的开发工作都迁移过来。当然也包括由 hexo 搭建的博客。

本以为直接将 Win 下由 `hexo init` 生成的文件夹复制过来，然后 `npm i` 就完事了。结果并不能如愿。当我用 npm 安装完 package ，完了之后，运行 `hexo g` 就报错了。

以下是报错信息：

```
Error: Cannot find module './build/Release/DTraceProviderBindings'

# 下面省略一长串
```

一开始还以为 npm 少装了什么模块。结果 `npm i` 了好多次。依然会报这个错。

# 解决方案

查了 hexo 在 github 上的 issue 。发现也有人遇到了相同的问题。

热心网友 MaShiZhao 的解决方案：

```
$ npm install hexo --no-optional
if it doesn't work
try
$ npm uninstall hexo-cli -g
$ npm install hexo-cli -g

```

试了第一个。结果就生效了。感谢这位老哥。

于是愉快地用 Mac 写下这篇博文。：）

# 本文参考

- [Hexo installed on Mac, throw error "Cannot find module './build/Release/DTraceProviderBindings'"](https://github.com/hexojs/hexo/issues/1922)