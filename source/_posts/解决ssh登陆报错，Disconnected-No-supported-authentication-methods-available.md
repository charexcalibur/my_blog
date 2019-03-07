---
title: '解决ssh登陆报错，Disconnected:No supported authentication methods available'
date: 2018-10-07 16:00:28
tags:
  - 阿里云
  - linux
  - ssh
thumbnail: https://cdn.axis-studio.org/IMG_6213.JPG
---
# 起因

昨天重启了自己的阿里云机器。重启之后 ssh 就连不上了。但是使用阿里云网页端的连接工具可以登陆。使用 mac 自带的 ssh 以及 filezilla 都报错 `Disconnected:No supported authentication methods available`。

上阿里云找了下文档。结果都是让重启机器。重启了两次，还是无法登陆。

只能 Google 了。查了下，可能是 ssh 配置的问题。根据方法是了一下，果然 ok 了。

# 解决

1. 通过阿里云网页端登陆 ecs
2. 进入 `etc/ssh/` 目录
3. 查看 `sshd_config` 内的 `PasswordAuthentication` 是否为 no 
4. 编辑 `PasswordAuthentication` 设置 `no` 为 `yes`
5. 输入命令 `service ssh restart`, 重启 ssh 服务

然后再通过 ssh 登陆机器。发现已经可以登陆了。

# 本文参考

- 封面摄自周浦花海

