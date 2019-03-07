layout: psot
title: JS引擎中的LHS，RHS查询
date: 2017-10-03 16:07:32
tags:
- JavaScript
- 作用域
categories:
- 笔记
thumbnail: https://cdn.axis-studio.org/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20171004112516.png
---

# 问题

```JavaScript
function foo(a) {
    var b = a;
    return a + b;
}

var c = foo(2);
```


1.找出其中所有的LHS查询。

2.找出其中所有的RHS查询。

## 何时进行RHS,LHS查询

>如果查找的目的是对变量进行赋值，那么就会使用LHS查询；如果目的是获取变量的值，就会使用RHS查询。

## 答案

- LHS查询3处
    - `var b = a;`中的对b进行赋值，需要LHS查询变量b
    - `var c = foo(2);`中对c进行赋值，需要LHS查询函数foo(2)调用后的值
    - `a = 2;`foo(2)中有一个隐式的对a的赋值操作

- RHS查询4处
    - `var b = a;`中获取参数a的值，需要进行RHS查询
    - `return a + b;`中需要分别对a和b进行RHS查询
    - `var c = foo(2);`中获取foo(2)的值，需要进行RHS查询


