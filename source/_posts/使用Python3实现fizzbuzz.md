---
title: 使用Python3实现fizzbuzz
date: 2017-05-28 16:12:57
tags:
- Python
categories:
- 技术
---
# fizzbuzz

玩法：打印1-100，当遇到数字为3的倍数时输出"fizz"，当遇到5的倍数时输出"buzz",当遇到既是3的倍数又是5的倍数时，输出"fizzbuzz"。<!--more-->

# 使用Python3实现

## 常用方法

```python
def fizzbuzz():
    for x in range(1,100):
        if x%3 == 0:
            if x%5 ==0:
                print("fizzbuzz")
            else:
                print("fizz")
        elif x%5 == 0:
            print("buzz")
        else:
            print(x)
fizzbuzz()
```

声明一个函数fizzbuzz(),在函数中使用一个for循环，在循环中对x进行余数判断，当x既是3的倍数又是5的倍数时`print("fizzbuzz")`，当x仅是3的倍数时`print("fizz")`，当x非3的倍数时，代码走到`elif`对x取5的余数，如果余数为0，则`print("buzz")`。否则`print(x)`。

这种写法比较常规，其他语言比如Java，JavaScript也基本是这个套路。但对于Python来说，这种写法不够优雅，不符合Zen of Python中的
> Simple is better than complex

接下来介绍一种优雅的写法。

## 优雅写法

```python
def fizzbuzz():
    for x in range(1,100):
        print("fizz"[x%3*4:] + "buzz"[x%5*4:] or x)

fizzbuzz()
```

在下第一次见到这种写法时，甚为吃惊。还有这种操作？我们来分析一波这个函数。
在for循环中使用`print()`来输出结果，在`print()`中使用`or`或语句来判断输出，
`or`之前使用两个字符串拼接输出。在下认为，这波操作，最骚的应该就是`"fizz"[x%3*4:]`这种对字符串进行切片操作了。当x为3的倍数时`x%3`的值为0，此时`[x%3*4:]`这个列表就是`[0:]`,此时将不会对字符串`"fizz"`进行切片操作，与此同时`"buzz"[x%5*4:]`中的`x%5`是会出现余数的，我们假设这个`x`的值就为3，那么`"buzz"[x%5*4:]`就该为`"buzz"[12:]`将会切去字符串的前12个字符，也就是说此时(x=3)的情况下，`print()`函数将会输出`fizz`。
