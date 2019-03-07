---
title: JS数据结构之JS实现链表
date: 2017-11-26 17:20:19
tags:
- 数据结构
- JavaScript
categories:
- 笔记
thumbnail: https://cdn.axis-studio.org/65966297_p0.png-min.jpg
---
# 开头

以本文来复习《学习JavaScript数据结构与算法》一书中链表相关知识。

## 何为链表

> 链表储存有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素由一个储存元素本身的节点和一个指向下一个元素的引用组成。

## 与数组的不同

对比数组，链表的好处在于：当添加或移除元素的时候不需要移动其他元素。

另一个不同点：数组可以直接访问任何位置的元素，而访问链表中的元素，需要从链表的head开始迭代列表直到找到所需的元素。

## 使用JavaScript实现链表

《学习JavaScript数据结构与算法》一书中的代码实现为ES5，本文将尝试使用ES6语法实现链表。

```js
function LinkedList() {
  class Node{
    constructor(element) {
      this.element = element
      this.next = null
    }
  }

  let length = 0
  let head = null

  this.append = element => {
    let node = new Node(element),
        current
    if (head === null){
      head = node
    } else {
      current = head

      // 循环列表，知道找到最后一项
      while(current.next){
        current = current.next
      }

      // 找到最后一项，将其next赋值为node，建立链接
      current.next = node
    }
    length ++
  }

  this.insert = (position, element) => {
    // todo
    // 检查越界值
    if (position >=0 && position <= length) {
      let node = new Node(element),
          current = head,
          previous,
          index = 0
      if (position === 0) {
        // 在第一个位置添加
        node.next = current
        head = node
      } else {
        while (index++ < position) {
          previous = current
          current = current.next
        }
        node.next = current
        previous.next = node
      }
      length++
      return true
    } else {
      return false
    }
  }

  this.removeAt = position => {
    // 检查越界值
    if(position > -1 && position < length) {
      let current = head,
          previous,
          index = 0

      // 移除第一项
      if (position === 0) {
        head = current.next
      } else {
        while (index++ < position) {
          previous = current
          current = current.next
        }

        // 将previous与current的下一项链接起来：跳过current，从而移除它
        previous.next = current.next
      }
      length--
      return current.element
    } else {
      return null
    }
  }

  this.remove = element => {
    let index = this.indexOf(element)
    return this.removeAt(index)
  }

  this.indexOf = element => {
    let current = head,
        index = 0

    while (current) {
      if (element === current.element) {
        return index
      }
      index ++
      current = current.next
    }
    return -1
  }

  this.isEmpty = () =>{
    return length === 0
  }

  this.size = () => {
    // todo
    return length
  }

  this.toString = () => {
    // todo
    let current = head,
        string = ''
    while (current) {
      string += ',' + current.element
      current = current.next
    }
    return string.slice(1)
  }

  this.getHead = () => {
    return head
  }
}
```

### Node辅助类

Node辅助类表示需要加入列表的项。包含：

- element属性，指将要添加到列表中的值
- next属性，一个指向下一个节点的指针

### 两个储存变量

- length属性，储存列表项的数量
- head属性，储存第一个节点的引用

### append方法

append方法向列表末尾添加一个元素，有两种场景：

- 列表为空，此时添加的是第一个元素
- 列表不为空，此时向列表追加元素

```js
  this.append = element => {
    let node = new Node(element),
        current
    if (head === null){
      head = node
    } else {
      current = head

      // 循环列表，知道找到最后一项
      while(current.next){
        current = current.next
      }

      // 找到最后一项，将其next赋值为node，建立链接
      current.next = node
    }
    length ++
  }
```

首先将实例化node。

当列表为空，也就是传入的元素为第一个元素时，将head指向node。

当列表不为空，要向列表尾部添加元素时，我们需要一个指向当前元素的`current`的变量。之后循环访问列表，直到`current.next`为`null`，此时表示已到达列表尾部。将`current.next`指向`node`就能实现在尾部添加元素了。

最后，每次`append`都会使实例的length+1。

### toString方法

先写toString方法可以有助于测试代码。

toString方法会将LinkedList对象转换成字符串。

```js
this.toString = () => {
    // todo
    let current = head,
        string = ''
    while (current) {
      string += ',' + current.element
      current = current.next
    }
    return string.slice(1)
  }
```

从head开始遍历每个元素，从头开始循环直到`current`为null，在循环中通过`,`拼接元素,同时指向先一个元素。最后返回字符串。

```js
// 测试
var list = new LinkedList()
list.append(11)
list.append(22)
list.toString() // "11,22"
```

[点我](https://codepen.io/charexcalibur/pen/rYJwrJ)，在线测试。

### removeAt方法

removeAt方法接收一个`position`来根据指定位置移除元素。

```js
  this.removeAt = position => {
    // 检查越界值
    if (position > -1 && position < length) {
      let current = head,
          previous,
          index = 0

      // 移除第一项
      if (position === 0) {
        head = current.next
      } else {
        while (index++ < position) {
          previous = current
          current = current.next
        }

        // 将previous与current的下一项链接起来：跳过current，从而移除它
        previous.next = current.next
      }
      length--
      return current.element
    } else {
      return null
    }
  }
```

首先需要验证这个`position`是否有效。无效时返回`null`。

当这个`position`有效时，需要从head开始，故将`head`赋值给`current`,同时也需要一个`previous`变量以及一个`index`索引。

当`position === 0`时，表示我们需要移除第一项，因为当前`current`元素就是`head`，所以需要将`head`通过`current.next`指向下一个元素。这样就会移除第一个元素。

当需要移除中间或者最后一个元素时，循环直到`index === position`,此时到达目标位置。将当前值`current`赋值给`previous`变量，将当前值的下一个`node`赋值给当前值。然后通过将previous与current的下一项链接起来：跳过current，从而移除它。

最后，每次`removeAt`都会使实例的length-1，并返回被删除的元素。

```js
// 测试
list.toString() // "11,22"
list.removeAt(0) // 11
list.toString() // "22"
```

[点我](https://codepen.io/charexcalibur/pen/rYJwrJ)，在线测试。

### insert方法

insert方法可以在任意位置插入一个元素。

```js
  this.insert = (position, element) => {
    // todo
    // 检查越界值
    if (position >=0 && position <= length) {
      let node = new Node(element),
          current = head,
          previous,
          index = 0
      if (position === 0) {
        // 在第一个位置添加
        node.next = current
        head = node
      } else {
        while (index++ < position) {
          previous = current
          current = current.next
        }
        node.next = current
        previous.next = node
      }
      length++
      return true
    } else {
      return false
    }
  }
```

同样的需要检查`position`是否有效。无效时返回`false`。

当有效时，将`element`实例化为`node`,其余变量设置与`removeAt`方法类似。

当在第一个位置添加元素时，将`node`的`next`指向`current`,同时`head`指向`node`。

若是其他场景，同样需要循环列表，找到目标位置。当跳出循环后，`node.next`指向当前元素，`previous.next`指向`node`。这样就在当`current`元素与`previous`元素之间插入了`node`。

最后每次插入元素列表的长度都需要+1，同时返回`true`作为信号。

```js
// 测试
list.toString() // "22"
list.insert(2,33) // false
list.insert(1,22) // true
list.toString() // "22,33"
list.insert(1,44) // true
list.toString() // "22,44,33"
```

[点我](https://codepen.io/charexcalibur/pen/rYJwrJ)，在线测试。

### indexOf方法

indexOf方法接收一个值，如果这个值存在于列表中，就返回元素的位置，否则就返回-1。

```js
  this.indexOf = element => {
    let current = head,
        index = -1 // 此处书中为 index = -1

    while (current) {
      if (element === current.element) {
        return index
      }
      index ++
      current = current.next
    }
    return -1
  }
```

变量的定义类似之前的方法。

循环止于`current`为null。

当元素为我们要找的元素时，返回`index`。否则就计数，检查下一个`node`。

```js
// 测试
list.toString() // "22,44,33"
list.indexOf(100) // -1
list.indexOf(22) // 0
```

[点我](https://codepen.io/charexcalibur/pen/rYJwrJ)，在线测试。

### 其余方法不再赘述

其余几个方法比较简单，故此处不再赘述。

本文所使用的链表类为单向链表。

其他种类链表将在之后补充。

## 本文参考

- 头图画师: [hakusai](https://www.pixiv.net/member.php?id=1589657)
- [《学习JaveScript数据结构与算法》](https://www.amazon.cn/s/ref=nb_sb_ss_ime_c_1_12?__mk_zh_CN=%E4%BA%9A%E9%A9%AC%E9%80%8A%E7%BD%91%E7%AB%99&url=search-alias%3Daps&field-keywords=%E5%AD%A6%E4%B9%A0javascript%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95&sprefix=%E5%AD%A6%E4%B9%A0JavaScript%2Caps%2C149&crid=2YLFLW9GCVLCP)
