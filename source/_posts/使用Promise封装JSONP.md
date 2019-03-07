---
title: 使用Promise封装JSONP
date: 2017-10-10 14:12:46
tags:
- JSONP
- 跨域解决方案
categories:
- 技术
thumbnail: https://cdn.axis-studio.org/JSONP.png
---
# 使用Promise对一个JSONP库进行封装

JSONP库来自[A simple JSONP implementation](https://github.com/webmodules/jsonp)

创建一个JS文件，引入安装过的jsonp库。将文件暴露成一个方法，这个方法接收三个参数:url, data, option

由于原始的JSONP库在引入url前需要将url拼好，而我希望在引入url时只引入不带参数的url地址，而将所有的参数都放在data里。option则对应原始库中的option。

```javascript
import originJSONP from 'jsonp'

export default function jsonp(url, data, option) {
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(data)
 
  return new Promise((resolve, reject) => {
    originJSONP(url, option, (err, data) => {
      if (!err) {
        resolve(data)
      } else {
        reject(err)
      }
    })
  })
}

function param(data) {
  let url = ''
  for (var k in data) {
    let value = data[k] !== undefined ? data[k] : ''
    url += `&${k}=${encodeURIComponent(value)}` // 拼接URL
  }
  return url ? url.substring(1) : '' // return时去掉&
}
```


# 本文参考
- [what-is-jsonp-all-about](https://stackoverflow.com/questions/2067472/what-is-jsonp-all-about)
- [A simple JSONP implementation](https://github.com/webmodules/jsonp)
- [JSONP的Promise封装](http://coding.imooc.com/class/chapter/107.html#Anchor)