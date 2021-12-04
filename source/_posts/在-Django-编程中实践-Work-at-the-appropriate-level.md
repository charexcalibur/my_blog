---
title: '在 Django 编程中实践  Work at the appropriate level '
date: 2021-09-15 18:36:56
tags:
  - Django
  - 技术
thumbnail:
  - https://cdn.axis-studio.org/bw2021/bw2021_5.jpg?imageMogr2/quality/50
---

# 记录一次 Django 性能优化

## 起因

今天收到反馈，测试环境一个接口很慢，从 Chrome 显示的响应时间将近 10s。

遂开始优化。

## 经过

首先使用 Django debug tool 来获取性能量化指标。发现一个 `limit=10, page=1` 的接口，出发了 2000ms 的 query，同时 cpu time 消耗了 7000ms，有些离谱。

将 Django setting 文件里的 debug 设置为 False，此时请求接口速度稳定在 2s 左右。

确定为使用了 **Django debug toolbar** 消耗了大部分性能，因为设置了当 debug 为 True 时，使用 debug toolbar。

继续测试。

我的方法是给该 filter 中的封装的函数做 timing。使用装饰器实现测试函数时间。

```python
def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        r = func(*args, **kwargs)
        end = time.perf_counter()
        print('handle {} items in {}.{} : {}'.format(r, func.__module__, func.__name__, end - start))
        return r
    return wrapper
```

根据打印信息得出，整体耗时 2s 多。显然很慢了。第一反应是考虑到因为有一些外键查询导致 `n+1`，采用 queryset 的 `select_related` 方法对外键查询进行优化。效果不明显。

review 代码，希望能找出具体耗时的代码逻辑。同样使用 start end 进行 timing，最终定位到下面这行代码：

```python
product_list = [item.product.id for item in queryset]
```

这行逻辑意图想要获取一个查询集中所有 product 外键的 id，于是写了一段列表推导。做了个 count 显示 list 长度 1w 多，显然问题出在这里。根据 Django 定义，此处触发大量查询，并 hit 到了数据库。

>  Work at the appropriate level

根据 Django 优化文档提示。不应该在服务层去做 product id 集合的查询，可以直接调用数据库方法来实现。

```python
product_list = queryset.values('product_id')
```

对新逻辑进行 timing，实际消耗时间 0.02s。

优化效果从 2s 到 0.02s，只改了一行代码，效果显著。

## 本文参考

- [Django 优化文档](https://docs.djangoproject.com/zh-hans/3.2/topics/performance/)
- 封面摄于 2021 bw