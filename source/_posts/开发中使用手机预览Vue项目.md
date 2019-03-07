---
title: 开发中使用手机预览Vue项目
date: 2018-06-30 12:47:20
tags:
  - Vue
  - 开发调试
  - 移动端
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/IMG_5919.JPG
---

# 目的

在开发 Vue 移动端项目时，我们经常需要使用移动端设备来测试项目效果。但目前本人使用的 Vue-cli 版本（2.9.1）并未支持 `run dev` 后移动端访问项目。

为了能得到更好开发效果，我决定自己来实现。

## 环境

- win 10
- Vue-cli 2.9.1
- Template version 1.2.7

## 思路

首先获取本地 ip 地址，在 webpack 配置中，当 `run dev` 时，修改 dev 的 host 配置为 ip 地址。然后通过手机访问该 ip 。实现在手机上预览项目的效果。

## 实施

配置文件的位置：to/your/product/config/index.js 。

首先，打开文件。追加以下代码。

```js
const os = require('os')

let IPv4

for (let i = 0; i < os.networkInterfaces().WLAN.length; i++) {
  if (os.networkInterfaces().WLAN[i].family === 'IPv4') {
    IPv4 = os.networkInterfaces().WLAN[i].address
    console.log(IPv4)
  }
}

console.log(`本地或手机打开 => ${IPv4}:8080`)

```

修改 dev 对象中的 host 属性。

```js
host: IPv4 || 'localhost'
```

最后在控制台中输入 `npm run dev` 启动项目。就可以在手机上输入 ip 地址访问项目了。

## 注意

可以发现读取的是 os 下的 WLAN 属性，如果使用的是以太网络则需要读取 以太网 。当然讲究点，可以写个判断，这样基本就可以做到通用了。

# 本文参考

- 封面来自爱车 Mazda Alexa 战损版
