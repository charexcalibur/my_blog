---
title: vue插件注册原理
date: 2019-06-06 18:08:04
tags: 
 - Vue.js
 - 源码分析
thumbnail: https://cdn.axis-studio.org/DSC00060.JPG?imageMogr2/quality/30
---


# vue 插件注册原理

Vue 使用了 `Vue.use()` 这个全局 api 来注册 Vue 插件。

`Vue.use()` 代码的路径在 vue 项目目录下的 `/src/core/global-api/use.js` 文件里。

```js
import { toArray } from '../util/index'

export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```
从源码中可以看出， `Vue.use` 是一个函数，接受一个插件（参数）。

这个函数首先会判断是否有注册的 `Plugins`, 如果有就赋值给变量 `installedPlugins` ，否则就定义为空值。

然后这个函数会先判断是传入的 `plugin` 参数是否已经存在于 `_installedPlugins`。如果存在，就直接 `return` ，避免重复注册。

接着将除了第一个插件之外的参数转换成 `array`, 并将 `this` (Vue)放到数组顶端。

再接着开始判断 `plugin` 是否有 `install` 这个函数，如果有就调用这个函数。

最后将 `plugin` 储存到 `installedPlugins`。

## 本文参考

- 封面图：迷之女主角
- [Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/)
