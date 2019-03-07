---
title: 记一次element-ui表单验证触发条件的踩坑
date: 2018-08-26 10:16:34
tags:
  - element-ui
  - vue
  - validator
  - trigger
thumbnail: https://cdn.axis-studio.org/IMG_6143.jpg
---

# 起因

因为要验证一个 输入建议 功能的输入框。所以使用了 element-ui 中的 autocomplete 这个组件。

autocomplete 中有一个事件，名为 `select` 表示在选中建议时触发。

同时由于业务的需要。还要对输入的内容进行校验。校验使用了自定义的 `validator` , 同时 trigger 选择了 `blur` （ 此处 trigger 的选择留下了一个坑 ）。

```javascript

  var validator_1 = (rule, value, callback) => {
    if (value === '') {
      callback(new Error('请不要输入空值'))
    } else if (index >1 ) {
      // 此处为另一个判断
      callback(new Error('该字段已存在'))
    } else {
      callback()
    }
  }

rules: {
  rule_1: [
    { validator: validator_1, trigger: 'blur' }
  ]
}
```

期望的结果是：当从选中建议时触发校验。然而结果是校验器会在 `select` 之前就对输入框进行校验。结果就是无法拿到正确的 `value` 从而一直报错 `请不要输入空值` .

# 解决方案

## 弯路

一开始想是否是自己在校验器中使用的 `setTimeout` 导致校验进入了异步，所以会出现检验不正确的现象。结果移除 `setTimeout` 后，依旧会出现上述问题。所以不是这个问题不是同步，异步的问题。

询问了同事，结果发现是 `trigger` 设置的问题。之前的 `trigger` 设置的是 `blur` ，也就是输入框失去焦点的时候触发。并不能用于 `select` 事件。

查了一下 MDN ， `select` 应该与 `change` 事件联用。

> change 事件被 `<input>`, `<select>`, 和 `<textarea>` 元素触发, 当用户提交对元素值的更改时。与  input 事件不同，change 事件不一定会对元素值的每次更改触发。

所以把 rules 中 `trigger` 改为 `change` 就 ok 了。

# 本文参考

- [event change](https://developer.mozilla.org/zh-CN/docs/Web/Events/change)
- [event blur](https://developer.mozilla.org/zh-CN/docs/Web/Events/blur)