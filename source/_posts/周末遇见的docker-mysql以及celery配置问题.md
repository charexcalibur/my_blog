---
title: 周末遇见的docker mysql以及celery配置问题
date: 2020-12-14 13:59:36
tags:
- Docker mysql
- Celery
- Centos
categories:
- 技术

thumbnail: https://cdn.axis-studio.org/lxx/DSC01840.jpg/thumbnail
---

# 周末遇见的 docker mysql 以及 celery 配置问题，以及解决思路

这周末解决了两个苦恼很久的 bug。问题都是小问题，解决方法也很简单，但是中间思路出现了问题，导致用了一天时间来解决。这里记录下来解决思路。避免踩坑。

## 问题一

- docker mysql 版本 8.0

这是一个在使用 docker mysql 时遇到的问题。我在 centos 上部署了一个 docker mysql。使用宿主机可以直接通过 mysql 命令连入。但是外网怎么都连接不成功，提示连接超时。

经过测试：

- 宿主机可以连入 docker mysql
- 外网通过 db 工具无法连接
- 安全组已开通端口

最后发现，问题出在防火墙。使用 iptables 把端口放开就可以了。

## 问题二

这是一个 celery 配置问题。

根据 django-celery 的配置。使用 `celery -A proj worker -l INFO` 可以正常启动 celery 并使用 redis 作为 broker。但是根据 [celeryd](https://docs.celeryproject.org/en/stable/userguide/daemonizing.html#init-script-celeryd) 的配置启动，log 中总是显示无法连接 pyamqp ，但是我明明设置的是 redis 作为 broker。最后发现是配置文件类型写错了。`/etc/default/celeryd` 这个文件得是 `celeryd.sh`。`cp celeryd celeryd.sh` 就可以了。重启 celeryd 就能正常启动 celery 了。


## 本文参考

- [celery daemonizing](https://docs.celeryproject.org/en/stable/userguide/daemonizing.html#init-script-celeryd)
- [django celery](https://docs.celeryproject.org/en/stable/django/first-steps-with-django.html)
- 封面图 摄于 2020 理想乡