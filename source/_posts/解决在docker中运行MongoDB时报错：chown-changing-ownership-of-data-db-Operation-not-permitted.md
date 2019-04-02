---
title: >-
  解决在 docker 中运行 MongoDB 时报错：chown: changing ownership of '/data/db': Operation not
  permitted
date: 2019-04-02 19:13:57
tags:
  - MongoDB
  - docker
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/AF9D13C8-BFDC-4111-930C-5D34A98DDF2F.jpg
---

# 解决在 docker 中运行 MongoDB 时报错：chown: changing ownership of '/data/db': Operation not permitted

## 环境

- macOS Mojave 10.14.3
- Docker version 18.09.2, build 6247962
- mongo tag 4

## 问题复现

   1. `docker pull mongo:4` 下载 MongoDB 镜像，版本是 4。
   2. `docker run --name mymongo -v /mymongo/data:/data/db -d mongo:4` 启动一个 mongo 容器，并命名为 mymongo，同时绑定映射文件夹为 /mymongo/data，试了一下 docker 并不会自动创建该文件夹，需要手动创建。
   3. `docker ps` 查看正在运行的容器，此时发现并没有容器运行。
   4. `docker logd mymongo` 报错 chown: changing ownership of '/data/db': Operation not permitted。

## 解决思路

1. 看报错应该是权限问题，所以给 /mymongo/data 加上了 everyone 权限，同时删除容器重新运行，但还是报同一错误。
2. 以为是命令权限问题，故删除容器，重新以 sudo 运行命令，结果还是不行

## 解决方案

  最后试了下把 /mymongo/data:/data/db 前面的路径改为自己目录下的路径，就可以了。
