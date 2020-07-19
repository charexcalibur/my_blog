---
title: 使用 select_related 进行 SQL 性能优化
date: 2020-06-17 22:26:59
tags:
 - Django
 - 性能优化
thumbnail: https://cdn.axis-studio.org/lg29.jpg
---

本周处理了一个远古接口的性能调优。

最终结果：

在 python manage.py runserver 环境下，query: 358 --> 252, query_time: 1528ms --> 2427ms

线上环境 300ms 左右。

## 优化过程

### 工具

使用 django-debug-toolbar 作为性能分析工具。服务开启 debug 模式。

首先请求一次接口，查看到 debug-toolbar 中的 SQL 耗时很高，同时存在大量 similar query 以及 duplicates。

根据 SQL 性能分析提供的分析数据。可以找到具体到哪一行的 ORM 操作了多少次。

经过观察发现有大量的外键查询。一般情况下由外键查询会产生 N+1 问题。这里可以使用 Queryset 自带的 select_related 来解决外键产生的 N+1 问题。

看一下官网文档的解释。

> Returns a QuerySet that will “follow” foreign-key relationships, selecting additional related-object data when it executes its query. This is a performance booster which results in a single more complex query but means later use of foreign-key relationships won’t require database queries.

举个例子吧。

```python
# before
query = OrderStatus.objects.filter(order__in=ord_org_list)
# after
query = OrderStatus.objects.filter(order__in=ord_org_list).select_related('order')
# 使用
```

这样在之后使用 queryset 调用外键的时候就可以减少一次查询。提高了性能。

当然，select_related 只适合一对多的关系。如果用到的字段是多对多，可以使用 prefetch_related。

## 本文参考

- 《Django 企业开发实战》
- [select_related 官网文档](https://docs.djangoproject.com/en/3.0/ref/models/querysets/)
- 封面图摄于临港南汇新城海滩