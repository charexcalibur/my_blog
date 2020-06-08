---
title: 深入理解 Javascript 中的 EventLoop
tags:
  - Javascript
thumbnail: 'https://cdn.axis-studio.org/bw_%20188.jpg'
date: 2020-05-04 00:23:57
---



这两天趁着休息又复习了一遍 Jake Archibald 在 JSConf 中关于 EventLoop 的[分享](https://www.youtube.com/watch?v=cCOL7MC4Pl0)。常看常新。想着，得写点东西记录下来。

## 主线程

首先，浏览器有一个主线程。在这里 JS 会被执行，DOM 会被渲染。因此所有的这些事件都会有一个顺序。

这也就意味着，当主线程上有任务需要很长时间去处理就会造成阻塞，页面就会出现卡顿。所以，除了主线程，我们还需要其他线程来帮忙处理网路请求、编码、解码、加密，监控设备输入等操作。但是，一旦这些线程需要页面的响应。此时，我们就需要通过某种途径去通知到主线程。

这种途径就是 EventLoop。EventLoop 将会来协调所有这些活动。

我们拿 `setTimeout` 来举个例子：

```javascript
setTimeout(callback, ms)
```

当 `setTimeout(callback, ms)` 被触发，就会运行以下步骤：

1. 离开当前线程并行执行以下步骤：
   1. 等待 `ms` 毫秒
   2. 生成一个队列去执行以下步骤：
      1. 执行 `callback` 函数

也就是说，当浏览器执行了 `setTimeout` 函数，此时就会离开当前线程，然后执行 `callback`。 但我们是在主线程之外的地方来触发这个回调函数的，假设这个回调函数是编辑 DOM 的话，这样可能会造成大量并行运行的 Javascript 编辑同一个 DOM。这样就会造成冲突。

所以我们会创建一个任务到任务队列，以便在未来的某个时间点，按照队列顺序回到主线程继续执行。

这就是浏览器原理的核心部分。

## Task Queues

在 EventLoop 之前，首先需要关注的是 Task Queues。

当没有任何事件时，EventLoop 只是在空转，以节省 cpu 消耗。

当我们向任务队列加入任务时，EventLoop 就会去处理他们，获取第一个 task 然后执行。

### The Render Steps

举个例子，我们在页面上放一个按钮。然后给按钮绑定一个点击事件触发一个无限循环。

```javascript
button.addEventListener('click', event => {
  while(true)
})
```

此时，当我们点击这个按钮，我们会告诉 EventLoop ，我们有一个 Task 需要执行。但是这个 Task 永远不会结束。此时页面就会卡住，即便我们告诉 EventLoop 我们有图片还需要渲染更新。

也就是说当我们执行以下代码时，页面会闪烁吗？

```javascript
// 页面会闪烁吗？
document.body.appendChild(el)
el.style.display = 'none'
```

答案是不会的，因为无论顺序如何，浏览器都会先将该任务执行完毕之后，才会去执行渲染。EventLoop 会确保你的任务会在下一次渲染之前执行完毕。

所以我们知道了 while 循环会阻塞渲染，因为这个任务永远也无法执行完毕，那么下面这个呢？

```javascript
function loop() {
  setTimeout(loop, 0)
}
loop()
```

这也是一个循环，在 `setTimeout` 中调用本身。此时 EventLoop 就会在每次事件循环中去执行这个 `loop()` 函数。线程并没有被阻塞，而是一直处于循环状态中。因此并不会阻塞渲染。

*如果你想执行渲染相关的代码，不要把它放在任务中。因为任务在渲染的另一边。*

## requestAnimationFrame

如果我们想要把渲染的代码放在渲染阶段之前，浏览器允许我们这样做，使用 requestAnimationFrame （RAF）可以做到这一点。

```javascript
function callback() {
  moveBoxForwardOnePixel()
  requestAnimationFrame(callback)
}
callback()
```

我们来移动一个盒子，每次我们将盒子向前移动一个像素，然后使用 `requestAnimationFrame` 来创造一个循环。

```javascript
function callback() {
  moveBoxForwardOnePixel()
  setTimeout(callback, 0) // 大约 4.7 ms
}
callback()
```

如果我们使用 `setTimeout` 来代替 `requestAnimationFrame` 会怎么样呢？我们可以打开 chrome console，执行下面的代码来对比一下。

```javascript
let node1 = document.createElement('div')
let node2 = document.createElement('div')
node1.setAttribute('id', 'box-1')
node1.setAttribute('style', 'position: relative;width:300px;height:300px;background-color: red;')
node2.setAttribute('id', 'box-2')
node2.setAttribute('style', 'position: relative;width:300px;height:300px;background-color: blue;')
document.body.appendChild(node1)
document.body.appendChild(node2)
let box1 = document.getElementById('box-1')
let box2 = document.getElementById('box-2')
let n = 0
let m = 0
box1.style.left = '1px'
box2.style.left = '1px'
function callback1(box, n) {
  n = n +1
  if(n<1000) {
    let left = box.style.left
    box.style.left = (parseInt(left) + 1) + 'px'
    requestAnimationFrame(() => callback1(box, n))
    }
}
function callback2(box, m) {
  m = m +1
  if(m<1000) {
    let left = box.style.left
    box.style.left = (parseInt(left) + 1) + 'px'
    setTimeout(() => callback2(box, m), 0)
    }
}
callback1(box1, n)
callback2(box2, m)
```

事实证明 box2 比 box1 的移动速度快了 3.5 倍。这意味着回调被更频繁的调用，这并不是一件好事。

到这我们都是使用的 `setTimeout` 来实现向任务队列添加任务。虽然并没有一种专门用来创建异步任务方法。但我们可以使用消息通道（message channel）来伪造它。Jake Archibald 对此作了测试。

经测试，我们每百分之二毫秒就得到一个任务。渲染可以发生在两个任务之间，而两次渲染之间则可以有成千上万个任务。

所以推荐使用 `requestAnimationFrame` 将动画打包起来，这样会减少很多重复的工作。

`requestAnimationFrame` 回调函数运行在浏览器处理 css 和绘制页面之前。所以像下面这样的代码可能看起来开销很大，在代码逻辑上，我们多次展示和隐藏一个盒子，但实际上并不大。

```javascript
button.addEventListener('click', () => {
  box.style.display = 'none'
  box.style.display = 'block'
  box.style.display = 'none'
  box.style.display = 'block'
  box.style.display = 'none'
  box.style.display = 'block'
  box.style.display = 'none'
  box.style.display = 'block' // 只有最后一行会起作用
})
```

在渲染发生之前，JavaScript 会将任务执行完。看看下面的代码，会发生什么。

```javascript
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'
  box.style.transform = 'translateX(500px)'
})
```

实际上 box 只会从 0 到 500。

```javascript
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'
  requestAnimationFrame(() => {
    box.style.transform = 'translateX(500px)'
  })
})
```

我们把第二部分动画放在 requestAnimationFrame 中。但动画与之前并没有发生变化。为什么呢？

原因是，用户点击了按钮，这是一个任务，所以我们来到了任务队列，执行 `transform` 和 `transition` 。然后我们加入了一个 AnimationFrame 并开始执行。此时会执行 `box.style.transform = 'translateX(500px)'` 然后开始渲染。所以动画还是老样子。

要想使动画如我们所期望的那样。我们需要先执行 requestAnimationFrame，然后等 `transform` 以及 `transition` 渲染完，然后再执行一次 requestAnimationFrame。这样就能在移动到 `1000px` 之后，再移动回 `500px` 了。

```javascript
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'
  requestAnimationFrame(() => {
    requestAnimationFrame(() => {
      box.style.transform = 'translateX(500px)'
    })
  })
})
```

也可以使用 `getComputedStyle(box).transform` 迫使浏览器更早地执行样式计算。

```javascript
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'
  getComputedStyle(box).transform
  box.style.transform = 'translateX(500px)'
})
```

## MicroTasks

看完了 requestAnimationFrame 我们接着来看看 MicroTasks。

MicroTasks 应该跟 Promise 关联起来理解。

```javascript
document.body.addEventListener('DOMNodeInserted', () => {
  console.log('stuff added to <body>!')
})
```

这段代码的逻辑是，我想在节点加入到 body 元素的时候得到通知。

```javascript
for(let i = 0; i < 100; i++>) {
  const span = document.createElement('span')
  document.body.appendChild(span)
  span.text.Content = 'Hello'
}
```

如果我们创建 100 节点，这将会产生多少事件呢？ 1 个？还是 100个？

答案是将会产生 200 个事件。100 个 span 产生 100 个事件，因为有 `span.text.Content = 'Hello'`，所以还会有额外 100 个事件产生来为 span 设置文本，并且冒泡。

MicroTasks 可能在任务中进行，也可能在渲染阶段作为 RAF 的一部分进行，任何 Javascript 运行的时候都可能执行微任务。

Promise 也使用了微任务。

```javascript
Promise.resolve().then(() => console.log('hey')) // 这里我们 queue 了一个 micro tasks
console.log('yo') // 先输出 yo， js 执行结束，然后执行微任务，输出 hey
```

那么我们页面上使用微任务创建一个无限循环会怎样呢？

```javascript
function loop() {
  Promise.resolve().then(loop)
}
loop()
```

同样的页面也卡死了。因为不断有 MicroTasks 被添加到队列。而 MicroTasks 队列中的任务不被消费完的话是不会继续执行 EventLoop 的。

我们前面提到了三种队列，Tasks、Animation callbacks，以及 MicroTasks。

1. Tasks， 每次只执行一个任务，如果有另一任务加进来，就会被添加到队尾。
2. Animation callbacks，动画回调会一直执行，直到队列中的所有任务都完成，如果动画回调中又有动画回调，它们会在下一帧执行。
3. MicroTasks，同样也是一直执行，直到队列为空。但是， 如果处理微任务的过程中有新的微任务加进来，加入的速度比执行快，那么就会永远执行微任务。EventLoop 会阻塞，直到微任务列表被完全清空，这就是它阻塞渲染的原因。

## 小测试

```javascript
button.addEventListener('click', () => {
  Promise.resolve().then(() => console.log('Microtask 1'))
  console.log('Listener 1')
})

button.addEventListener('click', () => {
  Promise.resolve().then(() => console.log('Microtask 2'))
  console.log('Listener 2')
})
```

我们在一个按钮上绑定两个一样的事件监听器，如果用户点击按钮，输出顺序是什么呢？



答案是：Listener 1,  Microtask 1,  Listener 2, Microtask 2

但如果用户使用 `button.click()` 结果就不一样了。

此时 js stack 中会先调用 `click()`，然后到 `Listener 1`，微任务排队，所以 log 出 `Listener 1`，此时 js stack 不为空, stack 中还有 `click()` 没被返回，所以，开始执行 `Listener 2`， 排队第二个微任务 `Microtask 2`,所以输出 `Listener 2`， 现在所有 js stack 清空，开始执行微任务。输出 `Microtask 1 Microtask 2` 。

所以答案是：Listener 1, Listener 2, Microtask 1, Microtask 2

## 本文参考

- [Jake Archibald: In the loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
- 封面摄于 BW-2019 coser: 鸪桑