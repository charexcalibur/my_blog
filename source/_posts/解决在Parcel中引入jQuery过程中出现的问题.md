---
title: 解决在Parcel中引入jQuery过程中出现的问题
date: 2018-02-16 23:15:23
tags:
  - Parcel
  - jQuery
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/QQ%E6%88%AA%E5%9B%BE20180128180019-min.png
---

尝试使用 Parcel 来打包项目。项目主要使用了 jQuery 和 Bootstrap 。以此文来记录期间遇到的问题以及解决方案。

# jQuery无法被正确引入

## 环境

- Node.js 8.9.1
- Parcel 1.5.1
- windows 10
- jQuery 3.3.1
- Bootstrap 3.3.7

## 初始化项目

- 安装 Parcel
- 安装 jQuery
- 安装 Bootstrap

创建 main.js

```javaScript
// main.js
import 'jquery' // 1 其实会报错
import 'bootstrap/dist/js/bootstrap.min.js'
import 'bootstrap/dist/css/bootstrap.min.css'

$(document).ready(function() {
  console.log('hello jquery')
})
```

运行 `parcel index.html`

### 期望效果

浏览器控制台输出 hello jquery

### 实际效果

浏览器报错 Bootstrap's JavaScript requires jQuery

证明 jQuery 并未被正确引入

## 解决方案

在项目目录的合适位置下新建文件 import-jquery.js

```javaScript
// import-jquery.js
import jquery from 'jquery'

export default (window.$ = window.jQuery = jquery)
```

```javaScript
// main.js
import '/your/path/import-jquery.js'
import 'bootstrap/dist/js/bootstrap.min.js'
import 'bootstrap/dist/css/bootstrap.min.css'

$(document).ready(function() {
  console.log('hello jquery')
})
```

此时浏览器控制台就会正确输出 hello jquery 了

# 本文参考

- 封面来自 [电影少女～VIDEO GIRL AI 2018～](http://www.zimuzu.tv/resource/35935)
- [How do I use jQuery and jQuery-ui with Parcel (bundler)?](https://stackoverflow.com/questions/47968529/how-do-i-use-jquery-and-jquery-ui-with-parcel-bundler)

