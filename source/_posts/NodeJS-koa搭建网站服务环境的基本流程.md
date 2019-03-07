---
title: NodeJS+koa搭建网站服务环境的基本流程
date: 2017-02-14 22:03:22
tags:
- NodeJS
categories:
- 技术
---
以此文来记录学习NodeJS以及其相关知识。

# 什么是koa

> koa是一个基于node环境的，更小、更富有表现力、更健壮的Web框架。


koa应用是一个包含一系列中间件generator函数的**对象**。这些中间件函数基于**request请求**以一个类似于栈的结构组成，并依次来执行。<!--more-->

koa的核心设计思路是为中间件提供高级[语法糖](https://www.zhihu.com/question/20651624?sort=created)封装,使得编写中间件变得有趣。

在kao中包含了content-negotiation（内容协商）、cache freshness（缓存刷新）、proxy support（代理支持）和redirection（重定向）等常用任务方法。与提供庞大的函数支持不同的是，koa只包含很小的一部分，因为koa并不绑定任何中间件。

```javascript
    var koa = require('koa');//require koa模块
    var app = koa();//新建koa实例

    app.use(function *(){
        this.body = 'Hello World';
    });//监听服务

    app.listen(3000);//监听服务在3000端口
```

# 基于nodeJS服务端的搭建和开发

## 工程目录结构

- mock 伪造的数据，（接口联调：服务端给定数据，根据数据格式，将数据渲染到页面上）
- node_modules 放npm的包
- service 文件夹下放服务相关的文件
	1. 返回数据，来自本地mock
	2. 对接口进行调用，调用线上真实的接口，返回的数据用来渲染页面
- static 文件夹下放css,img,script等静态资源和静态文件
- view  将要开发项目的模板文件
- app.js 项目启动的入口文件

## 使用koa中间件构建网站雏形

在项目根目录下，新建app.js文件作为整个项目的入口。在node环境下，在命令行中输入node app.js来启动项目。

```javascript
    var koa = require('koa');
    var controller = require('koa-route');
    var app = koa();

    //使用app.use()来调用中间件
    app.use(controller.get('/route_test',function*(){
        this.set('Cache-Control','no-cache');//设置页面不缓存

        this.body = 'hello koa！';//body是返回体，返回的是html内容，可以是文本也可以是内容

    }));//koa接收的第二个函数不是callback函数，而是一个generator，相当于一个异步执行的函数

    app.listen(3001);
```

在命令行中输入`node app.js`启动项目，在地址栏中输入127.0.0.1:3001/route_test，页面将会显示hello koa！

以上就成功的搭建了一个最简单的web服务器，通过url和固定的route返回特定的内容。

## 如何渲染模板

```javascript

    var koa = require('koa');
    var controller = require('koa-route');
    var app = koa();
    var views = require('co-views');//具有模板渲染能力的中间件
    var render = views('./view',{
        map : { html :'ejs' }  //指定模板渲染类型，接收map的key值，key值中选择模板引擎
    });//实例化,调用这个方法，返回一个对象，接收两个参数，第一个是模板的存放路径，

    app.use(controller.get('/route_test',function*(){
        this.set('Cache-Control','no-cache');
        this.body = 'hello koa！';

    }));

    app.use(controller.get('/ejs_test',function*(){
        this.set('Cache-Control','no-cache');
        this.body = yield render('test',{title:'title_test'});
    }));


    app.listen(3001);
```

`this.body = yield render('test',{title:'title_test'});`

render()接收两个参数，第一个为模板的名字，“{}”为传递到模板中的参数。

新建test.html文件，在test.html文件中写代码`<%=title%>`。

这里的“=”号代表数据绑定，绑定的是title这个字段。



如何理解[yield](https://www.zhihu.com/question/30258965)。

来[这里](http://es6.ruanyifeng.com/)了解ES6。

> co-views是对通用模板引擎渲染平台consolidate的封装。根据对co-views源码的分析，它把consolidate统一的api有封装成return function(done){...}的形态，这样源代码中的yield render（view,model）就能够融入generator的逻辑之中。

总的来说，是使用koa的co-views中间件来渲染模板生成render实例。

## 如何访问静态文件

```javascript

    var koa = require('koa');
    var controller = require('koa-route');
    var app = koa();
    var views = require('co-views');
    var render = views('./view',{
        map : { html :'ejs' }
    });
    var koa_static = require('koa-static-server');

    app.use(koa_static({
        rootDir:'./static/',//整个项目工程的目录，注意前面有"."
        rootPath:'/static/',//在url地址里访问静态文件目录用的路径
        maxage : 0 //文件的过期时间，缓存
    }));


    app.use(controller.get('/route_test',function*(){
        this.set('Cache-Control','no-cache');
        this.body = 'hello koa！';

    }));

    app.use(controller.get('/ejs_test',function*(){
        this.set('Cache-Control','no-cache');
        this.body = yield render('test',{title:'title_test'});
    }));


    app.listen(3001);
```


以上就基本搭建好了一个网站的框架，可以访问静态资源文件，有路由来指定访问内容，同时有模板渲染。

接下来介绍如何调用后端服务的接口。

## 为网站添加Mock接口

在service目录下，新建service.js。该文件并不直接提供数据，而是将后端提供的原始数据进行处理，转换成为json的数据格式，最后传递给前端。

### 模拟数据

在mock目录下新建test.json文件，存放测试数据

    {
        msg: 0,
        data:'test data'
     }

测试数据一般有两个域，一个是状态域msg，用来标明返回数据状态，另一个是data域标明数据类型和格式。

点击[这里](http://www.w3school.com.cn/json/index.asp)了解JSON

当数据格式为mock时，使用以下代码来将数据暴露成接口。

```javascript

    var fs = require('fs');//使用mock格式的文件时，需要访问文件系统，使用node中的fs模块


    exports.get_test_data = function() {
        var content = fs.readFileSync('./mock/test.json','utfd-8');
        //读取test.json文件
        return content;
        //把数据暴露出来
    }
 ```

暴露出来的数据需要访问的接口，接口在app.js中暴露。在app.js中写

```javascript

    var service = require('./service/service.js')

    app.use(controller.get('api_test',function*(){
        this.set('Cache-Control','no-cache');
        this.body = service.get_test_data();
    }));
```

### 线上接口的调用

在service.js中

```javascript

    exports.get_search_data = function(start,end,keyword){
        return function(cb){
            var http = require('http');
            var qs = require('querystring');//将object转换成http的查询参数
            //{a : '1'}转换成http://127.0.0.1/api?=1
            var data = {
                s: keyword,
                start: start,
                end: end
            };

            var content = qs.stringify(data);//将data转换成http的查询参数
            var http_request = {
                hostname: '',//主机地址
                port: 80, //指定端口
                path: '' +　content//接口路由的位置
            }

            //发送请求
            req_ojb = http.request(http_quest,function(_res){
                var content = '';
                _res.setEncoding('utf8');

                //触发data事件后返回chunk
                _res.on('data',function(chunk){
                    content +＝ chunk；
                });

                //监听end事件，当触发end事件后才会返回所有数据
                _res.on('end',function(){
                    cb(null,content);
                });
            });

            //监听error事件
            req_obj.on('error',function(){

            });
            //发送请求
            req_obj.end();
        }
    }

之后在app.js中调用，先加入一个路由接口

       app.use(controller.get('/ajax/search',function*(){
            this.set('Cache-Control','no-cache');
            var querysting = require('querystring');
            var params = querystring.parse(this.req._parseUrl.query);
            var  start = params.start;
            var  end = params.end;
            var  keyword = params.keyword;
            this.body = yield service.get_search_data(start,end,keyword);
        }));
```

到此就是使用NodeJS+koa搭建网站服务环境的大致流程。










# 本文参考了

- 《[koa文档](http://koa.bootcss.com/#)》
- 《[用 Koa 写服务体验](http://www.open-open.com/lib/view/open1434961473669.html)》
- 《[NodeJS 模块系统](http://www.runoob.com/nodejs/nodejs-module-system.html)》





