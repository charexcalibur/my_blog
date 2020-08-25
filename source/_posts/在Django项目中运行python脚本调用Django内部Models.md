---
title: 在 Django 项目中运行 python 脚本调用 Django 内部的 Models
date: 2020-08-06 14:20:24
tags:
  - Django
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/cp26/cp26day156.jpg
---

业务需求是实现一个数据同步脚本，要求对接 qiniu，从 qiniu 获取资源，处理后存到 Mysql 中。

## 问题

原本以为很简单的逻辑，结果被 Django 摆了一道。

```python
# 部分代码
from qiniu import Auth, BucketManager
from moerank.common.models import CoserInfo, CoserNoPic
# 业务逻辑
```

结果疯狂报错 `ModuleNotFoundError: No module named 'moerank'`。

## 解决解决思路

怎么都找不到 Django 项目的 module。我寻思着，不能吧，尝试：

- 修改 setting 文件中的 install_app, 无效
- 根据官网文档 [calling-django-setup-is-required-for-standalone-django-usage](https://docs.djangoproject.com/en/3.0/topics/settings/#calling-django-setup-is-required-for-standalone-django-usage) 在脚本中引入 django.setup(), 无效

到此想到了，大概率是环境变量的问题。也就是说在运行脚本的时候，需要将项目地址加入到 `sys.path` 中。

## 解决方案
```python
# 部分代码
from qiniu import Auth, BucketManager
from moerank.common.models import CoserInfo, CoserNoPic

sys.path.append('/path/to/your/django_product')
os.environ['DJANGO_SETTINGS_MODULE'] = 'django_product.settings'

django.setup()

# 业务逻辑
```
将项目路径加入到 `sys.path` 中运行脚本，脚本正常运行。

## 为什么

我打印了原始 `sys.path`

```python
[
  '/Users/hayato/Mywork/moerank/moerank',
  '/Users/hayato/mywork/moerank'
]
```

第一项是项目初始化添加到 `sys.path` 中的路径，第二项是在脚本中手动添加的项目路径。也就是说项目初始化时绑定的路径到 project 。也就是说 `sys.path` 中的路径必须是跟 `manage.py` 同级。而不能是 django project 路径。

查了下老项目的 `sys.path` 确实如此。

那么为什么会出现新开的项目需要手动配置 `sys.path` 呢？

这个项目我是根据 DRF 官网的 quickstart 初始化的。我想问题应该出在初始化的那几个命令上。

```shell
# Create the project directory
mkdir tutorial
cd tutorial

# Create a virtual environment to isolate our package dependencies locally
python3 -m venv env
source env/bin/activate  # On Windows use `env\Scripts\activate`

# Install Django and Django REST framework into the virtual environment
pip install django
pip install djangorestframework

# Set up a new project with a single application
django-admin startproject tutorial .  # Note the trailing '.' character
cd tutorial
django-admin startapp quickstart
cd ..
```

直觉告诉我 `django-admin startproject tutorial .` 问题很大。

### 测试

1. `mkdir sys_test && python3 -m venv env && . env/bin/activate` 初始化 env 并进入
2. `python` 进入 python shell
3. ```shell
   >>> improt sys
   >>> print(sys.path)
   ['', '/usr/local/Cellar/python/3.7.2_2/Frameworks/Python.framework/Versions/3.7/lib/python37.zip', '/usr/local/Cellar/python/3.7.2_2/Frameworks/Python.framework/Versions/3.7/lib/python3.7', '/usr/local/Cellar/python/3.7.2_2/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload', '/Users/hayato/Mywork/sys_test/env/lib/python3.7/site-packages']
   ```
   打印出来初始化的 `syspath`
4. `pip install django && pip install djangorestframework`
5. `django-admin startproject tutorial .`
6. 查看 `sys.path` 并没有变化，直觉出现了错误。继续走下面的命令
7. `cd tutorial &&
django-admin startapp quickstart`
8. 查看 `sys.path` 并没有变化。 也就是说 `sys.path` 并不是在项目初始化的时候做的绑定。那么就可能是在项目运行的时候绑定的。

不该从 python shell 中去查看 `sys.path` ， 因为我新建一个 py 脚本，从脚本中打印 `sys.path`，项目路径已经在里面了。

删除 test 项目，重新开始测试。在初始化 env 之后直接新建 py 脚本打印 `sys.path` 发现项目路径已经出现在 `sys.path` 中了。

## 无法复现

奇怪。
