---
title: 使用Vue计算属性实时验证输入
date: 2017-06-17 16:21:21
tags:
- Firebase
- Vue
categories:
- 技术
---

这个案例实现了客户端实时数据同步，以及在输入的时候通过计算属性实时验证。<!--more-->

# 准备

- [Vue.js](https://cn.vuejs.org/) 2.3.3
- [Firebase bindings for Vue.js](https://github.com/vuejs/vuefire)

本项目已托管在[Github](https://github.com/charexcalibur/vue-example/blob/master/email_verification/index.html)
在线地址[email_verification](https://charexcalibur.github.io/vue-example/email_verification/index.html)

## 代码

```html
<div id="app">
	<ul is="transition-group">
		<li v-for="user in users" class="user" :key="user['.key']">
			<span>{{user.name}} - {{user.email}}</span>
			<button v-on:click="removeUser(user)">X</button>
		</li>
	</ul>
	<form id="form" v-on:submit.prevent="addUser">
		<input type="text" v-model="newUser.name" placeholder="Username">
		<input type="email" v-model="newUser.email" placeholder="email@email.com">
		<input type="submit" value="Add User">
	</form>
	<ul class="errors">
		<li v-show="!validation.name">Name cannot be empty.</li>
		<li v-show="!validation.email">Please provide a valid email address.</li>
	</ul>
</div>
```

```javascript
    let emailRE = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;

    // Setup Firebase
    // Initialize Firebase
	const config = {
         apiKey: "AIzaSyAZc5fVRwzPxeV-oVLyyEkRvwfnYU-AFV4",
		 authDomain: "excalibur-01.firebaseapp.com",
		 databaseURL: "https://excalibur-01.firebaseio.com",
	 	 projectId: "excalibur-01",
	 	 storageBucket: "excalibur-01.appspot.com",
	 	 messagingSenderId: "688446613330"
	};
	firebase.initializeApp(config);


	let db = firebase.database();
    let usersRef = db.ref('users');

    // create Vue app
    let app = new Vue({
        // element to mount to
        el: '#app',
        // initial data
        data: {
            newUser: {
                name: '',
                email: ''
            }
        },
        // firebase binding
        // https://github.com/vuejs/vuefire
        firebase: {
            users: usersRef
        },
        // computed property for form validation state
        computed: {
            validation: function (){
                return {
                    name: !!this.newUser.name.trim(),//双叹号强制转换为布尔值
                    email: emailRE.test(this.newUser.email)
                }
            },
            isValid: function () {
                let validation = this.validation;
                return Object.keys(validation).every(function (key) {
                    return validation[key]
                })
            }
        },
        // methods
        methods: {
            addUser: function () {
                if (this.isValid) {
                    usersRef.push(this.newUser);
                    this.newUser.name = '';
                    this.newUser.email = '';
                }
            },
            removeUser: function (user) {
                usersRef.child(user['.key']).remove()
            }
        }
    })
```

### computed属性

`computed`中的`validation`属性会根据`newUser`返回一个Object，这个Object在`isValid`属性中被赋值给变量`validation`。

在属性`isValid`中，`Objectkeys(validation)`返回一个数组，对这个数组进行`every()`测试

```javascript
    function (key) {
        return validation[key];
     }
```

当`validation`中的`name`和`email`的值都是true时，以上语句会返回true。

### methods属性

methods中的两个方法都是vuefire api中常用方法。具体可以查文档。

## 本文参考了

- [vuefire文档](https://github.com/vuejs/vuefire)
- [Firebase保存数据](https://firebase.google.com/docs/database/web/save-data#basic_write)