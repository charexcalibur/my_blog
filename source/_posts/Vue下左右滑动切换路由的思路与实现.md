---
title: Vue下左右滑动切换路由的思路与实现
date: 2017-11-19 18:35:48
tags:
- Vue
categories:
- 基础组件开发
thumbnail: https://cdn.axis-studio.org/65331605_p0%20%282%29.jpg
---

# 起因

在使用Vue开发单页面应用程序的时候，希望能实现左右滑动来实现tab的切换。

由于未能找到现成的实现，决定自己写一个。

## 思路

关键思路：使用touch事件，在触发事件时，`push`路由，同时使用Vue的动画过渡效果，最终实现目的。

## 工具

Vue下有封装好的touch事件插件--[vue-touch](https://github.com/vuejs/vue-touch)。此文写下时，该插件还未支持Vue2.0，但可以使用`vue-touch@next`版本。

使用了Vuex来管理`routerIndex`。因为如果没有一个索引来标识路由的话，在点击tab实现跳转后，再使用滑动来跳转时，取得路由状态有点麻烦，所以索性使用Vuex来管理状态，这样在`app.vue`以及`tab.vue`下都可以随时取得路由索引。

## 实现

[在线地址](http://music.axis-studio.org/#/recommend)

```html
<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <v-touch @swipeleft="goRight" @swiperight="goLeft">
      <transition name="slide-fade">
        <keep-alive>
          <router-view></router-view>
        </keep-alive>
      </transition>
    </v-touch>
    <player></player>
  </div>
</template>
```

```javascript
import Vue from 'vue'
import MHeader from 'components/m-header/m-header'
import Tab from 'components/tab/tab'
import Player from 'components/player/player'
import VueTouch from 'vue-touch'
import router from './router'
import {mapGetters, mapMutations} from 'vuex'

Vue.use(VueTouch, {name: 'v-touch'})

export default {
  data() {
    return {
      transitionName: 'slide-left',
      pathArr: []
    }
  },
  computed: {
    ...mapGetters([
      'currentPathIndex'
    ])
  },
  methods: {
    ...mapMutations({
      setCurrentPathIndex: 'SET_CURRENT_PATH_INDEX'
    }),
    goRight() {
      let pathObj = this.$router.options.routes
      let nextIndex = this.currentPathIndex + 1
      this.setCurrentPathIndex(nextIndex)
      if (nextIndex >= 5) {
        nextIndex = 4
        this.setCurrentPathIndex(4)
      }
      // let currentPath = this.$router.currentRoute.path
      router.push(pathObj[nextIndex])
    },
    goLeft() {
      let pathObj = this.$router.options.routes
      let nextIndex = this.currentPathIndex - 1
      this.setCurrentPathIndex(nextIndex)
      if (nextIndex <= 1) {
        nextIndex = 1
        this.setCurrentPathIndex(1)
      }
      router.push(pathObj[nextIndex])
    }
  },
  components: {
    MHeader,
    Tab,
    Player
  }
}
```

```stylus
  .slide-fade-enter-active
    transition all .3s ease
  .slide-fade-leave-active
    transition all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0)
  .slide-fade-enter, .slide-fade-leave-to
    transform translateX(10px)
    opacity 0
```

`<v-touch>`标签包裹`router-view`, 在触发`v-touch`的`swipeleft`以及`swipeRight`事件时，添加对应的回调函数来实现路由的切换。

## 本文参考

- 头图画师[木shiyo](https://www.pixiv.net/member.php?id=40222)
- [慕课网Vue-music](http://coding.imooc.com/class/107.html?mc_marking=7a72c833ff9ae725588c7c13fe7d2f96&mc_channel=banner)
- [Vue-router](https://router.vuejs.org/zh-cn/)