---
title: element+express+mongoose实现分页查询
date: 2017-12-29 14:53:20
tags:
- Element
- Nodejs
- MongoDB
categories:
- 业务实现
thumbnail: https://cdn.axis-studio.org/66474729_p0-min.jpg
---

一个element配合express实现分页查询的思路与实现。

# 技术栈

- Element
- Express
- Mongoose

## 前端实现

前端的分页使用了Element的[分页组件](http://element-cn.eleme.io/#/zh-CN/component/pagination)。

```html
  <el-pagination
    @current-change="handleCurrentChange"
    :current-page="currentPage"
    :page-size="pageSize"
    layout="total, prev, pager, next"
    :total="count">
 </el-pagination>

 <script>
  export default {
    data () {
      return {
        count: 0,
        currentPage: 1,
        pageSize: 10
      }
    },
    methods: {
      // 获取当前页码并重新获取数据
      handleCurrentChange (val) {
        this.currentPage = val
        this._initData()
      },
      // 初始化数据，获得数据数量以及每页数据
      _initData () {
        let param = {
          currentPage: this.currentPage,
          pageSize: this.pageSize
        }

        axios.get('to/your/api').then((response) => {
          let res = response.data
          if (res.status === '0') {
            this.count = res.result.count
          }
        })

        axios.get('to/your/api', {
          params: param
        })
        .then((response) => {
          let res = response.data
          if (res.status === '0') {
            this.questionsList = res.result.list
          } else {
            this.questionsList = []
          }
        })
      }
    }
 }
 </script>
```

这里使用了两个api，一个用来获取数据总数，一个根据页码来获取该页数据。

## 后端实现

后端使用express+mongoose实现对数据的分页查询。

```js
// 后端关键代码
router.get("/show", (req, res, next) => {
  let Questions = require('../models/questions') // 引入models
  let currentPage = parseInt(req.param('currentPage')) // 转换前端传入当前页码
  let pageSize = parseInt(req.param('pageSize')) // 转换前端传入的每页大小
  let skip = (currentPage-1)*pageSize // 实现分割查询的skip
  let params = {}

  // 使用mongoose的skip,limit两个api对数据进行跳跃查询并返回查询结果
  let questionsModel = Questions.find(params).skip(skip).limit(pageSize)
  questionsModel.exec((err, doc) => {
    if (err) {
      res.json({
        status: '1',
        msg: err.message
      })
    } else {
      res.json({
        status: '0',
        msg: '',
        result: {
          list: doc
        }
      })
    }
  })
})
```

# 本文参考

- 封面画师[Novelance](https://www.pixiv.net/member.php?id=10710834)
- ELement[文档](http://element-cn.eleme.io/#/zh-CN/component/pagination)
- mongoose[中文文档](https://mongoose.shujuwajue.com/guide/models.html)