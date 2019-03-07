---
title: 解决使用Django搭建后台系统，输入中文报错问题
date: 2017-04-19 20:17:14
tags:
- Python
- Django
categories:
- 技术
---
## 问题描述

平台：ubuntu

Python版本：2.7

Django：1.11<!--more-->

项目实例：《Python编程从入门到实践》2017年2月第2次印刷，项目3，Web应用

问题描述：
- 使用 Python 建立虚拟环境
- 在虚拟环境环境中，使用Django创建项目
- 跟书本走到18.2.6，在文本框中输入中文，点击save，报错

报错内容：

`'ascii' codec can't encode characters in position 0-2: ordinal not in range(128)`

## 解决方法

找出manage.py文件，在文件里添加代码：

```python

import sys

reload(sys)
sys.setdefaultencoding('utf8')

```

在项目的文本框中输入中文，点击 save ，可以添加中文文本。