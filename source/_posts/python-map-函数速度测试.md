---
title: python map 函数速度测试
date: 2020-08-25 13:53:55
tags:
  - python
thumbnail: https://cdn.axis-studio.org/cj/cj15.jpg
---

Python 版本 3.7.9

分别使用 map+lambda、列表推导式以及 for 循环，对列表中每个元素乘以 2 ，最终返回一个列表。

```shell
# map lambda
python -m timeit -s 'xs=range(1000000)' 'list(map(lambda x: x*2, xs))'
2 loops, best of 5: 119 msec per loop

# 列表推导
python -m timeit -s 'xs=range(1000000)' '[x*2 for x in xs]'
5 loops, best of 5: 76.6 msec per loop

# for 循环
python -m timeit -s 'xs=range(1000000)' 'l = []' 'for i in xs: l.append(i*2)'
2 loops, best of 5: 112 msec per loop
```

根据结果可以看出，使用列表推导更快。