---
title: 工具函数之insertArray函数
date: 2017-10-31 21:16:46
tags:
- ES6
- Util
categories:
- 工具函数
thumbnail: https://cdn.axis-studio.org/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20171031190949-min.png
---

目的是为了处理这样一个需求：一个显示历史搜索的组件，显示最大条数为15条的历史信息，每次搜索后，将搜索的key显示在第一条，若该key之前被搜索过，则移除之前的key，将本次的添加到第一条。

# insertArray实现

```
function insertArray(arr, val, compare, maxLen) {
  const index = arr.findIndex(compare)
  if (index === 0) {
    return
  }
  if (index > 0) {
    arr.splice(index, 1)
  }
  arr.unshift(val)
  if (maxLen && arr.length > maxLen) {
    arr.pop()
  }
}
```

函数接收4个参数：
- arr：被操作的数组
- val：被比较的值
- compare：一个执行比较规则的函数
- maxLen：规定数组的最大长度

该函数首先会根据compare函数来获取元素的索引，一般`(item) => {return item === val}`，当`index === 0`,也就是说该元素就是arr中的第一条时，直接return；当元素存在且不是第一条时，将元素从数组中移除，并将val移动到数组首位；当存在长度限制且数组长度大于限制长度时，将数组最后的元素移除。