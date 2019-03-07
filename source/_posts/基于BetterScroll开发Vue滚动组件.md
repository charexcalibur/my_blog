---
title: 基于BetterScroll开发Vue滚动组件
date: 2017-10-16 15:17:08
tags:
- Vue.js
- 组件
categories:
- 基础组件开发
thumbnail: https://cdn.axis-studio.org/VueScroll-min.png
---
# 目的

在SPA开发过程中，我们经常会遇到需要实现列表滚动的需求。为了方便，我们通常将列表滚动的组件开发成一个基础组件，在业务组件中需要时来使用。<!--more-->

# 开始
## 创建组件
首先我们需要创建一个名为`scroll.vue`的组件，并引入BetterScroll。
```Vue
<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>

<script>
import BScrool from 'better-scroll'
</script>
```

## 父子组件通信
```Vue
<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>

<script>
import BScrool from 'better-scroll'

export default {
  props: {
    probeType: {
      type: Number,
      default: 1
    },
    click: {
      type: Boolean,
      default: true
    },
    data: {
      type: Array,
      default: null
    },
    listenScroll: {
      type: Boolean,
      default: false
    }
  },
  mounted() {
    setTimeout(() => {
      this._initScroll()
    }, 20)
  },
  methods: {
    _initScroll() {
      if (!this.$refs.wrapper) {
        return
      }
      this.scroll = new BScrool(this.$refs.wrapper, {
        probeType: this.probeType,
        click: this.click
      })
    },
    enable() {
      this.scroll && this.scroll.enable()
    },
    disable() {
      this.scroll && this.scroll.disable()
    },
    refresh() {
      this.scroll && this.scroll.refresh()
    },
    scrollTo() {
      this.scroll && this.scroll.scrollTo.apply(this.scroll, arguments)
    },
    scrollToElement() {
      this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments)
    }
  },
  watch: {
    data() {
      setTimeout(() => {
        this.refresh()
      }, 20)
    }
  }
}
</script>
```

通过props的方式将BetterScroll的一些控制参数暴露给父组件:
- probeType: 选择派发scroll事件的时机
- click: 点击列表是否派发click事件
- data: 列表数据
- listenScroll: 是否派发滚动事件

更多参数见[文档](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/#better-scroll 是什么)

## 初始化BetterScroll
在Vue实例mounted时，设置timeOut来确保dom渲染完毕后调用`_initScroll`来初始化Betterscroll。

通过使用Vue的`$refs`来获得scroll的dom，BetterScroll提供一个类，实例化的第一个参数是dom，第二个参数是配置。

我们在methods对象里声明一个私有方法`_initScroll`来初始化BetterScroll。

为了确保数据变化后滚动功能依然正常，需要在watch对象中监听data的变化。

## 封装完毕

至此就完成了在Vue框架中对BetterScroll的封装。使用时，父组件只要将data通过prop传给scroll组件就可以实现正常滚动了。还有一些其他功能也可以将BetterScroll的一些方法封装在scroll中，通过prop来控制就可以了。



# 本文参考
- [BetterScroll](https://ustbhuangyi.github.io/better-scroll/doc/en/#the-implementation-of-better-scroll-in-mvvm-frameworks)
- [Vue开发音乐App实战](http://coding.imooc.com/class/chapter/107.html#Anchor)