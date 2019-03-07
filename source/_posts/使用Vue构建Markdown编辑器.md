---
title: 使用Vue构建Markdown编辑器
date: 2017-06-04 15:17:43
tags:
- Vue.js
categories:
- 技术
---


写本文的目的是为了梳理Vue的相关知识。<!--more-->


# 准备

- [Vue.js](https://cn.vuejs.org/) 2.3.3
- [marked.js](https://github.com/chjj/marked)
- [underscore.js](http://underscorejs.org/)

本项目代码已托管在[Github](https://github.com/charexcalibur/vue-example/blob/master/markdown_editor/index.html)

在线地址[markdown_editor](https://charexcalibur.github.io/vue-example/markdown_editor/index.html)<!--more-->

## 代码分析

```html
<div id="editor">
    <textarea :value="input" @input="update"></textarea>
    <div v-html="compiledMarkdown"></div>
</div>
```

```html
    <script>
            new Vue({
                el: '#editor',
                data: {
                    input: '# hello'
                },
                computed: {
                    compiledMarkdown: function () {
                        return marked(this.input, { sanitize: true })
                    }
                },
                methods: {
                    update: _.debounce(function (e) {
                        this.input = e.target.value
                    }, 300)
                }
            })
    </script>
```

代码省略了样式部分以及marked.js和underscore.js的引入。

先从`html`结构来看，页面分成左右结构。左边为markdown编辑区，右边为markdown编译区。

编辑区用`<textarea>`标签来定义。在标签中通过`v-bind:`(缩写为`:`)绑定value属性，同时通过`v-on`(缩写为`@`)绑定事件监听，`v-on:`需要接收一个定义的方法来调用，此处的`update`正是在下面的`methods`属性中定义的。

通过`v-on:`指令来监听文本框的输入，获取内容的函数作为`_.debounce`参数，另一个作为时间参数的值为`300`，而输入的值会用户停止操作的300m后作为参数传给`marked()`然后被编译成`html`格式再返回。

所以本人认为`<textarea>`中的`value`属性负责返回文本框的内容，然后Vue实例中的`update`将输入的`value`传给`compiledMarkdown`属性中的方法，该方法返回经过编译的`html`。

以上大概就是vue制作markdown编辑器的大概分析。

本人刚开始接触Vue.js，对其功能实现的理解难免有误，若您在阅读本文时发现问题，可以在文章下方的评论中指出，也可以通过邮件将问题发送给我，我会第一时间处理，感谢。

### 本文参考

- [浅谈 Underscore.js 中 _.throttle 和 _.debounce 的差异](https://blog.coding.net/blog/the-difference-between-throttle-and-debounce-in-underscorejs)
- [marked](https://github.com/chjj/marked)
