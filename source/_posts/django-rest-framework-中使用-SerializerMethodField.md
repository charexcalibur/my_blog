---
title: django rest framework 中使用 SerializerMethodField
date: 2019-08-11 15:39:18
tags:
  - django
  - python
  - django rest framework
thumbnail: https://cdn.axis-studio.org/EAiKqWFWkAEJ8ti.jpeg
---

在使用 `django rest framework` 生成接口时，可能会遇到这种情况。有一个 `a` 接口的 `serializer` ，我们想从 `serializer` 提供的接口中获取更多的字段信息。这种情况下，我们可以用 DRF 提供的 `SerializerMethodField` 方法。

## SerializerMethodField

官方对 SerializerMethodField 的解释。

***This is a read-only field. It gets its value by calling a method on the serializer class it is attached to. It can be used to add any sort of data to the serialized representation of your object.***

当我们需要使用当前 `model` 中字段去查询其他 `model` 时，可以使用这个方法。

## 例子

首先我们有一个 `serializer` class，需求是，我们需要用 `AlphaModel` 中的 `a_id` 字段去查 `BetaModel` 中的 `b_name`。并将结果添加到 `AlphaSerializer` 的 `fields` 中。

我们需要写一个 `class Bname` 继承 `serializers.Serializer`， 来序列化查询来的 `queryset`。记得当时查出来的是一个 `orderedDict` 所以要使用下面代码中的方式才能那大数据。

`serializers.py`

```pyhton

from rest_framework import serializers

class BnameSerializer(serializers.Serializer):
    b_name = serializers.CharField()

class AlphaSerializer(serializers.ModelSerializer):

    b_name = serializers.SerializerMethodField()

    class Meta:
        model = AlphaModel
        fields = [
          'id',
          'a_id',
          'a_name',
          'b_name'
        ]

    def get_b_name(self, obj):
        querySet = BetaModel.objects.filter(a_id=obj.a_id)
        data = BnameSerializer(instance=querySet, many=True)
        return data.data[0]['b_name']
```

## 本文参考

- 封面图: GOODSMILE 初音未来 AMG GT3
- [django rest framework docs](https://www.django-rest-framework.org/api-guide/fields/)