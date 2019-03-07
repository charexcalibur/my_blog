---
title: 解决elementUI导航点击同一个导航路由不刷新的问题
date: 2018-10-10 22:12:39
tags:
  - Vue-router
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/IMG_0607.jpg
---

# 起因

今天接了一个需求，在需要在点击路由一个相同的路由链接时也要更新页面数据。查了一下，当路由地址未发生改变时，router 是不会触发更新的，因为 url 没有发生变化。

但是需求还是要做啊。不然感觉体验不好。

下面就来讲一下我的思路。

# dependencies version

这次的解决方案是基于下面的依赖版本。

- element-ui ^2.4.5
- vue ^2.3.3
- vue-router ^2.6.0

# 思路

其实上面已经提到了，经过测试 router 没有触发更新的原因是 url 没有发生变化。所以我第一个想到的办法就是手动给路由添加一个唯一的 query 参数。

# 解决方案

很简单，两步就搞定了。

1. 写一个 `function` 监听 `click` 事件。
2. 在这个 `function` 中使用 `this.$router.push` 跳转路由，在传入 `path` 的同时传入 `query` , `query` 中添加一个时间戳字段。

这样就能在点击相同路由时，`url` 后会跟一串表示唯一性的时间戳，确保了 `url` 的唯一性。

函数可以这样写。

```js
clickItem () {
  let path = this.$route.path
  this.$router.push({
    path,
    query: {
      t: + new Date()
    }
  })
}
```

# 本文参考

- [
手摸手，带你用vue撸后台 系列三(实战篇)
](https://segmentfault.com/a/1190000009762198#articleHeader3)
- [vue-router issues#296](https://github.com/vuejs/vue-router/issues/296)