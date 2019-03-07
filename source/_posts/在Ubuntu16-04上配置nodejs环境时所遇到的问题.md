---
title: 在Ubuntu16.04上配置nodejs环境时所遇到的问题
date: 2017-09-26 17:30:09
tags:
- Ubuntu16.04
- Node.js
categories:
- 技术
---

在服务器上正确安装nodejs并配置软连接后，使用node命令提示node为正确安装。<!--more-->

# 环境

- Ubuntu 16.04
- nodejs 6.11.3

# 问题

在服务器上正确安装nodejs并配置软连接后，使用node命令提示node未正确安装。

![](https://cdn.axis-studio.org/QQ%E5%9B%BE%E7%89%8720170926181305.png)
![](https://cdn.axis-studio.org/QQ%E5%9B%BE%E7%89%8720170926181245.png)

一开始使用的是v8以上的新版本，考虑是否是新版本的问题。结果重装LTS版本后依然提示node未正确安装。

# 一种解决方案

尝试通过将node，npm的路径添加到PATH来解决上述问题。

`vi /etc/profile`

编辑配置文件。

在PATH行根据格式将node的路径添加进去。

添加成功后，保存并退出。执行`source /etc/profile`。

之后输入node，此时应该就在命令行中使用node了，npm同理。