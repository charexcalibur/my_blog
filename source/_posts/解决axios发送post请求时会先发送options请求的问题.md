---
title: 解决axios发送post请求时会先发送options请求的问题
date: 2018-04-14 16:19:50
tags:
  - post
  - options
  - 跨域
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/IMG_5484.JPG
---


# 解决 axios 发送 post 请求时会先发送 options 请求的问题

在开发时使用 axios 发送 post 请求时，发现在发送 post 请求之前会先发送一个 options 请求。

## 环境

- 前端
  - Vue.js  v2.5.2
  - Element UI
  - axios v0.17.1
- 后端
  - express v4.15.2

## 问题

当从前端发送 post 请求后，会先发送一个 options 请求

![](https://blog.axis-studio.org/QQ%E6%88%AA%E5%9B%BE20180414210505.png)

![](https://blog.axis-studio.org/QQ%E6%88%AA%E5%9B%BE20180414210548.png)

## 原因

> `OPTIONS` requests are what we call `pre-flight` requests in `Cross-origin resource sharing (CORS)`.

就是说在跨域请求时，浏览器会出在正式通信之前发送一个预检请求。

## 解决方案

增加一个请求头信息  `application/x-www-form-urlencoded`。

```js
  let config = {
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  axios.post('http://localhost:3000/questions/update', params, config)
  .then((response) => {
    console.log(response)
    this.idPass()
  })
```

这样再次使用发送 post 请求时，就不会再出现 options 请求了。

然而我并不觉得有必要移除这个 `pre-flight`。

# 本文参考

- 封面摄于上海顾村公园 2018 春
- [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
- [Why is an OPTIONS request sent and can I disable it?
](https://stackoverflow.com/questions/29954037/why-is-an-options-request-sent-and-can-i-disable-it)
