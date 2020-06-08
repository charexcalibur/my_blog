---
title: 在 Django Restframework 的 Serializer 中实现动态 fields
tags:
    - Python
    - Django
thumbnail: https://cdn.axis-studio.org/bw_%2017.jpg
---

在做数据权限的时候，我们可能会碰到根据用户权限对一个接口动态显示字段的需求。

这个需求我们可以在数据序列化的时候实现。具体实现还是看代码吧。

## 动态 fields 实现

```python
from rest_framework import serializers

class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = '__all__'

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        context = kwargs.get('context', None)
        if context:
            request = kwargs['context']['request'] # 获取 request 参数
            field_name_list = [item.field_name for item in request.user.role.field_config] # 根据用户角色获取 field_list

            if field_name_list:
                allowed = set(field_name_list)
                existing = set(self.fields)
                for field_name in existing - allowed:
                    self.fields.pop(field_name)
```

思路很简单，在序列化类初始化的时候，根据用户角色获取对应 field 配置，然后将 fields 中不允许显示的 pop 出去。最终就会得到该用户角色权限。

## 本文参考

- (django-rest-framework-serializer)[https://www.django-rest-framework.org/api-guide/serializers/]
- 封面摄于 BW-2019 coser: 鸪桑