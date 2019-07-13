---
title: ts-axios中的解析header逻辑
date: 2019-07-13 20:56:52
tags:
  - typescript
  - ts-axios
  - 源码分析
thumbnail: https://cdn.axis-studio.org/ORG_DSC00214.jpg?imageMogr2/quality/30
---

最近在跟着黄老师的学使用 `typescript` 重构 `axios` 。在学习源码的过程的同时，也学到了一些平时写代码的技巧。故记录下来。

## 处理响应 header

需求就是将通过 `XMLHttpRequest` 的 `getAllResponseHeaders` 方法获得的字符串解析成对象。

字符串：

```
date: Fri, 05 Apr 2019 12:40:49 GMT
etag: W/"d-Ssxx4FRxEutDLwo2+xkkxKc4y0k"
connection: keep-alive
x-powered-by: Express
content-length: 13
content-type: application/json; charset=utf-8
```

规则是每一行以 `\r\n` 结尾。最终希望成解析成下面的对象结构：

```json
{
  date: 'Fri, 05 Apr 2019 12:40:49 GMT'
  etag: 'W/"d-Ssxx4FRxEutDLwo2+xkkxKc4y0k"',
  connection: 'keep-alive',
  'x-powered-by': 'Express',
  'content-length': '13'
  'content-type': 'application/json; charset=utf-8'
}
```


实现 `parseHeaders` 工具函数

```js
export function parseHeaders(headers: string): any {
  let parsed = Object.create(null) // 创建一个空对象
  if (!headers) { // 如果没有 headers 直接返回空对象
    return parsed
  }

  // 如果有 headers 则按回车符、换行符分割成数组，此时数组为每一行的字符串
  headers.split('\r\n').forEach(line => {
    let [key, val] = line.split(':') // 对每一行的字符串以 : 分割成 key 和 val
    key = key.trim().toLowerCase() // 对 key 和 val 进行处理，去除空格并转换成小写
    if (!key) { // 
      return
    }
    if (val) {
      val = val.trim()
    }
    parsed[key] = val // 将 key 和 val 添加到 parsed 对象中
  })

  return parsed // 最终返回 parsed 对象
}
```

## 本文参考

- 封面图：摄于塘桥人行天桥下，命名为 启程
- 源码来自黄老师的使用 `typescript` 重构 `axios`