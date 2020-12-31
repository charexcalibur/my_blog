---
title: 这周使用 Django 遇到的两个问题及解决方案
date: 2020-12-30 21:28:17
tags:
  - Django
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/lxx/DSC01374.jpg
---


前段时间养成了一个好习惯，解决一个问题之后立马写个备忘录来记录，然后整理成博客。希望成坚持住。

这周记录了两个在使用 Django 时遇到的问题。一个是 `circular import`, 一个是 `model get_or_create()`。

## circular import 问题

遇见这个问题是因为准备将 model save 中的一些逻辑移动到 celery 中处理。在写 task 的时候，由于逻辑中需要回写一些表数据的状态需要引入该表的 model ，然而这个 task 也要引入到该表的 model 中。所以就会出现 django circular import 问题。

原因是我在写 task 时，在文件开头引入了 model，这样做并不可取。应该将引入 model 的逻辑写入 task 函数内。

比如：

```python
from django.apps import apps

def celery_task():
  model = apps.get_model(app_label='', model_name='')
  do_some_thing()
```


## get_or_create return 2 的问题

业务中遇到了 get_or_create  return 2 的情况。其实 django 官网有一段 warning

> Warning
>
> This method is atomic assuming that the database enforces uniqueness of the keyword arguments (see unique or unique_together). If the fields used in the keyword arguments do not have a uniqueness constraint, concurrent calls to this method may result in multiple rows with the same parameters being inserted.

就是说如果关键字参数中使用的字段没有唯一性约束，则对该方法的并发调用可能会导致插入多个具有相同参数的行。

所以要在 model 中给关键字段加上 unique 或者 unique_together。


## 参考

- [Django model querysets get-or-create](https://docs.djangoproject.com/en/3.1/ref/models/querysets/#get-or-create)
- [Cannot import models into Celery tasks in Django](https://stackoverflow.com/questions/57666355/cannot-import-models-into-celery-tasks-in-django)