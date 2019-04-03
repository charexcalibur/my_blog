---
title: MongoDB学习笔记--创建文档
date: 2019-04-02 20:57:16
tags:
  - MongoDB
categories:
  - 笔记
---

# MongoDB学习笔记--创建文档

最近打算系统的学一下 MongoDB 。计划写一些笔记把学习过程记录下来。

## 文档主键

MongoDB 主键的值存储在 _id 这个字段里。

- 文档主键具有唯一性
- 支持所有数据类型（数组除外）
- 复合主键

如果创建文档时不提供主键，MongoDB 就会自动创建一个 ObjectId 主键。

- 默认的文档主键
- 可以快速生成的 12 字节 id
- 包含创建时间

## 创建文档

`use test`，使用 test 数据库

`show collections`，查看 test 数据库中的合集

### 创建单个文档

创建单个文档命令

``` shell
db.<collection>.insertOne({
  <document>,
  {
    writeConcern: <document>
  }
})

```
这里的 `<collection>` 将替换成想要写入的集合的名字，`<document>` 替换成想要写入的文档， `writeConcern` 定义了本次文档创建操作的安全写级别。

返回
```json
{
  "acknowleged": true,
  "insertedId": "yourId" // 这个 id 是你插入的文档 _id
}
```
acknowleged 表示安全写级别被启用。

### 创建多个文档

创建多个文档命令

```shell
db.<collection>.insertMany({
  [<document1>,<document2>,...]
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
})
```

ordered 表示是否要按顺序写入文档，当设为 false 时，会以乱序写入，提高性能。

### 创建一个或者多个文档

创建一个或者多个文档

```shell
db.<collection>.insert({
  <document or array of documents>,
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
})

db.<collection>.save({
  <document>,
  {
    writeConcern: <document>
  }
})
```

当 save 处理一个新文档时就会调用 insert() 命令。



