---
title: 使用 Python 实现 LRU 缓存策略
date: 2021-07-31 17:17:22
tags:
  - python
  - 算法
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/bw2021/bw2021_14.jpg?imageMogr2/quality/50
---

了解一下 LRU 的缓存策略。

前几天听朋友在讨论 LRU。平时只知道 redis 中有相关缓存策略的配置。几个人讨论的热火朝天，我却无法插嘴。赶快补一下原理。



# LRU（Least Recently Used） Cache

字面意大概是最久未使用的缓存将会被淘汰掉。

## 使用 哈希表 + 双向链表 实现

查了一下 leetcode ，题解中使用 哈希表 + 双向链表来实现 LRU。

其中：
  - 哈希表：使用 `dict` 来做哈希映射，缓存 `key` 和对应节点。
  - 双向链表：使用双向链表的顺序来表示数据是否被最近使用过。靠近头部的是最近使用的，靠近尾部的是最久未被使用的。

对于 `get put` 操作，会先判断 `key` 是否存在于哈希表，然后再根据规则判断去操作链表。访问哈希表有对应的 `key`, 时间复杂度为 `O(1)`, 操作链表的时间复杂度也为 `O(1)`。所以这算法应该是一种 *空间换时间* 的做法。

> 在双向链表的实现中，使用一个伪头部（dummy head）和伪尾部（dummy tail）标记界限，这样在添加节点和删除节点的时候就不需要检查相邻的节点是否存在。


下面是大概的流程图：

![](https://cdn.axis-studio.org/blog/LRU%E6%B5%81%E7%A8%8B%E5%9B%BE.svg)

配合 leetcode 上的代码逻辑：

```python

class DLinkedNode:
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None


class LRUCache:

    def __init__(self, capacity: int):
        self.cache = dict()
        # 使用伪头部和伪尾部节点
        self.head = DLinkedNode()
        self.tail = DLinkedNode()
        self.head.next = self.tail
        self.tail.prev = self.head
        self.capacity = capacity
        self.size = 0

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        # 如果 key 存在，先通过哈希表定位，再移到头部
        node = self.cache[key]
        self.moveToHead(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key not in self.cache:
            # 如果 key 不存在，创建一个新的节点
            node = DLinkedNode(key, value)
            # 添加进哈希表
            self.cache[key] = node
            # 添加至双向链表的头部
            self.addToHead(node)
            self.size += 1
            if self.size > self.capacity:
                # 如果超出容量，删除双向链表的尾部节点
                removed = self.removeTail()
                # 删除哈希表中对应的项
                self.cache.pop(removed.key)
                self.size -= 1
        else:
            # 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
            node = self.cache[key]
            node.value = value
            self.moveToHead(node)
    
    def addToHead(self, node):
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node
    
    def removeNode(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def moveToHead(self, node):
        self.removeNode(node)
        self.addToHead(node)

    def removeTail(self):
        node = self.tail.prev
        self.removeNode(node)
        return node

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/lru-cache/solution/lruhuan-cun-ji-zhi-by-leetcode-solution/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

## 本文参考

 - 封面摄于 bw2021
 - [LeetCode-Solution](https://leetcode-cn.com/problems/lru-cache/solution/lruhuan-cun-ji-zhi-by-leetcode-solution/)