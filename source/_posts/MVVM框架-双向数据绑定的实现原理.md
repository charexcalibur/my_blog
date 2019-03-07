---
title: MVVM框架-双向数据绑定的实现原理
date: 2017-12-16 20:46:52
tags:
- MVVM框架
categories:
- 笔记
thumbnail: https://cdn.axis-studio.org/66292640_p0.jpg
---

也用Vue写了几个项目了，对双向数据绑定的实现很好奇。故查阅了一些资料和源码来了解双向数据绑定的实现。

Github上有一个的[关于双向绑定的实现](https://github.com/DMQ/mvvm
)。本文则是阅读此篇文章之后的个人理解。

# 实现的关键

> vue是通过数据劫持的方式来做数据绑定的，其中最核心的方法便是通过Object.defineProperty()来实现对属性的劫持，达到监听数据变动的目的。

要实现双向数据绑定，就必须要实现以下几点：

- 实现数据监听器Observer,用来对数据对象中所有的属性进行监听，并在数据变动时发出通知。

- 实现指令解析器Compile,对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。

- 实现Watcher,作为Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。

- 最后一个是MVVM的入口函数。

具体的实现可以看[DMQ](https://github.com/DMQ/mvvm)在Github上的实现。

## 主要是Object.defineProperty()api的使用

在实现Observer的时，需要利用`Obeject.defineProperty()`来监听属性的变动。需要对数据对象，包括子属性对象的属性进行递归和遍历，并都加上`setter`和`getter`。这样，当给这个对象的某个值赋值的时候，就会触发`setter`，就可以监听到数据的变化了。

代码来自[DMQ](https://github.com/DMQ/mvvm)。

```js
var data = {name: 'oldname'}

observe(data)

data.name = 'newname' // 监听到变化

function observe(data) {
  if(!data || typeof data !== 'object') {
    return
  }

  // 取出所有的属性遍历
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive(data, key, val) {
  observe(val) // 监听子属性
  Object.defineProperty(data, key, {
    enumerable: true, // 可枚举
    configurable: false, // 不能再define
    get: () => val,
    set: newVal => {
      console.log('监听到值的变化了', val, '-->', newVal)
      val = newVal
    }
  })
}
```

## 本文参考

- 封面画师[wlop](https://www.pixiv.net/member_illust.php?mode=medium&illust_id=66292640)
- [DMQ](https://github.com/DMQ/mvvm)