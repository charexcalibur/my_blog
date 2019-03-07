---
title: 使用Vue展示最新的Vue.js提交数据
date: 2017-06-11 12:46:37
tags:
- Vue.js
categories:
- 技术
---

本文目的是通过官网的第二个示例，学习Vue的生命周期、计算属性、列表渲染等基础操作。<!--more-->

# 准备

- [Vue.js](https://cn.vuejs.org/) 2.3.3


本项目已托管在[Github](https://github.com/charexcalibur/vue-example/blob/master/vue_commits/index.html)

在线地址[Vue_commits](https://charexcalibur.github.io/vue-example/vue_commits/index.html)
<!--more-->


## 代码分析

```html
<div id="demo">
	<h1>Latest Vue.js Commits</h1>
	<template v-for="branch in branches">
		<input type="radio"
			   :id="branch"
			   :value="branch"
			   name="branch"
			   v-model="currentBranch">
		<label :for="branch">{{ branch }}</label>
	</template>
	<p>vuejs/vue@{{ currentBranch }}</p>
	<ul>
		<li v-for="record in commits">
			<a :href="record.html_url" target="_blank" class="commit">{{ record.sha.slice(0, 7) }}</a>
			- <span class="message">{{ record.commit.message | truncate }}</span><br>
			by <span class="author"><a :href="record.author.html_url" target="_blank">{{ record.commit.author.name }}</a></span>
			at <span class="date">{{ record.commit.author.date | formatDate }}</span>
		</li>
	</ul>
</div>
```

```javascript
    let apiURL = 'https://api.github.com/repos/vuejs/vue/commits?per_page=3&sha=';

	let demo = new Vue({
		el: '#demo',

		data: {
			branches: ['master', 'dev'],
			currentBranch: 'master',
			commits: null
		},

        // created 这个钩子在实例被创建之后被调用

		created: function(){
			this.fetchData()
		},

		watch: {
		    currentBranch: 'fetchData'
		},

		filters: {
		    truncate: function (v) {
				let newline = v.indexOf('\n');
				return newline > 0 ? v.slice(0, newline) : v;
            },
			formatDate: function (v) {
		        return v.replace(/T|Z/g, ' ');
			}
		},

		methods: {
		    fetchData: function () {
		        let xhr = new XMLHttpRequest();
		        let self = this;
		        xhr.open('GET', apiURL + self.currentBranch);
		        xhr.onload = function () {
					self.commits = JSON.parse(xhr.responseText);
					console.log(self.commits[0].html_url);
                };
                xhr.send();
            }
		}
	})
```


`<script>`中实例化了一个demo的Vue实例，该实例挂载在`#demo`上。

当实例被创建后，通过`created`这个钩子函数调用实例中的`fetchData`函数。`watch`属性来响应数据变化，此处当`currentBranch`的值发生变化后，会调用`methods`中的`fetchData`函数。在`fetchData`函数中，向api地址发送请求，请求地址后拼接了当前选择的分支。在`xhr.onload`中将`responseText`赋值给实例中的`commits`。

实例中还定义了过滤器`filters`，过滤器被用在列表标签中的mustache插值中，对`commits`返回的值进行过滤。过滤器将`|`之前语句的值传入过滤器对应的函数。


## 本文参考了
- [HTML5 template标签元素简介](http://www.zhangxinxu.com/wordpress/2014/07/hello-html5-template-tag/)
- [Watchers](https://cn.vuejs.org/v2/guide/computed.html)