---
title: JavaScript原型链研究与面向对象
date: 2017-11-12 21:20:55
tags:
- JavaScript
categories:
- 笔记
thumbnail: https://cdn.axis-studio.org/Assassin%27s%20Creed%C2%AE%20Origins2017-11-7-0-22-29.jpg
---
# 开头

首先我们需要了解`prototype`和`__proto__`这两个名词。

## 什么是`prototype`?

> 每一个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

简而言之，`prototype`是函数所固有的属性。

## 什么是`__proto__`?

前面提到过，当我们创建一个函数时，会根据一定规则为改函数创建一个`prototype`属性，这个属性指向函数的原型对象。

> 默认情况下，所有原型对象都会自动获得一个`constructor`属性，这个属性包含一个指向`prototype`属性所在函数的指针。

当我们调用构造函数创建一个新的实例后，该实例会包含一个内部的指针(属性)，指向构造函数的原型对象，这个指针就是`__proto__`。

## 来看点代码

```javascript
function Person() {}
Person.prototype.name = 'hayato'
Person.prototype.sayName = function() {
  console.log(this.name)
}

var person1 = new Person()
var person2 = new Person()

console.log(Person.prototype) // {name: "hayato", sayName: ƒ, constructor: ƒ}
console.log(Person.prototype.constructor) //ƒ Person() {}
console.log(person1.__proto__) // {name: "hayato", sayName: ƒ, constructor: ƒ}
console.log(person2.__proto__) // {name: "hayato", sayName: ƒ, constructor: ƒ}
console.log(Person.prototype === person1.__proto__) // true
console.log(Person.__proto__) // ƒ () { [native code] }
console.log(Person.__proto__ === Object.__proto__) // true
console.log(Person.__proto__.__proto__ === Function.prototype.__proto__) // true
console.log(Person.__proto__.__proto__.__proto__ === Object.prototype.__proto__) // true

// 有点乱。。。我们来理一理
console.log(Function.__proto__ === Function.prototype) // true
console.log(Function.prototype.__proto__ === Object.prototype) // true
console.log(Object.__proto__ === Function.prototype) //true
console.los(Object.__proto__ === Function.__proto__) // true
```

> 最后我们可以看到`Function.__proto__`指向自己,而`Function.prototype.__proto__`指向的是`Object.prototype`,而`Object.__proto__`指向了`Function.prototype`,`Function.prototype`则等于`Function.__proto__`。

正好论证了前面提到的`__proto__`指向构造函数的原型对象。

## 构造函数、原型对象、实例的关系

构造函数通过`new`来生成实例。

构造函数的`prototype`指向原型对象。

实例的`__proto__`指向原型对象。

原型对象的`constructor`指向构造函数。

## 利用原型链实现继承

```javascript
// 根据之前的代码
console.log(person1.sayName()) // hayato
console.log(person2.sayName()) // hayato
```

在`Person`的原型对象上加入的方法或属性可以被实例共享。这就是原型链的工作方式。

## new运算符的工作原理

当使用`new`运算符时:

1. 一个新对象被创建，继承`foo.prototype`。
2. 构造函数foo被执行。执行的时候，相应的参数会被传入，同时上下文的`this`会被指定为这个新的实例。`new foo`等于`new foo()`，只能用于不传参的情况。
3. 如果构造函数返回一个对象，那么这个对象会取代整个`new`出来的结果。如果构造函数没有返回对象，那么`new`出来的结果为第一步中创建的对象。

```javascript
var new1 = function(func) {
  var o = Object.create(func.prototype)
  var k = func.call(o)
  if (typeof k === 'object') {
    return k
  } else {
    return o
  }
}

// 验证
var o = new1(Person) // Person {}

o instanceof Person // true
Person.prototype.walk = function() {
  console.log('walk')
}

console.log(o.walk())// walk
```

# 面向对象

## 类的声明

在线测试，[点我](https://codepen.io/charexcalibur/pen/dZWpLE)。

```javascript
// 类的声明
function Animal() {
  this.name = 'name'
}

// ES6声明
class Animal2 {
  constructor () {
    this.name = 'name'
  }
}

// 实例类的对象
console.log(new Animal(), new Animal2())
```

## 继承

### 借助构造函数实现继承

```javascript
// 借助构造函数实现继承
function Parent1() {
  this.name = 'parent1'
}
Parent1.prototype.say = function() {}
function Child1() {
  Parent1.call(this) // 将父级this指向子构造函数的实例
  this.type = 'child1'
}
console.log(new Child1())
```

原理：`call()`方法使子构造函数中的this指向父构造函数，从而使子构造函数可以调用父构造函数中的方法。

缺点: `Child1`无法继承`Parent1`原型对象上方法，只能继承构造函数中的内容。

### 借助原型链实现继承

```js
// 借助原型链实现继承
function Parent2() {
  this.name = 'parent2'
}
function Child2() {
  this.type = 'child2'
}
Child2.prototype = new Parent2()
console.log(new Child2)
```

原理：使`new`出来的`child2.__proto__`等于`Parent2`的实例。从而使`child2`实例继承`Parent2`。

缺点：所有实例引用一个原型对象，在实例中改变原型对象中的属性或方法，其他的实例中的属性或方法也会跟着改变。

### 组合方式实现继承及优化

```js
//组合方式实现继承
function Parent3() {
  this.name = 'parent3'
  this.play = [1, 2, 3]
}
function Child3() {
  Parent3.call(this)
  this.type = 'child3'
}
Child3.prototype = new Parent3()

let s3 = new Child3()
let s4 = new Child3()

// 组合继承的优化1
function Parent4() {
  this.name = 'parent4'
  this.play = [1, 2, 3]
}
function Child4() {
  Parent4.call(this)
  this.type = 'child4'
}
Child4.prototype = Parent4.prototype
let s5 = new Child4()
let s6 = new Child4()
console.log(s5, s6)

console.log(s5 instanceof Child4, s5 instanceof Parent4)
console.log(s5.constructor)
```
优化1，通过使用`Parent4.call(this)`在`Child4`实例中得到`Parent4`中的内容，之后通过`Child4.prototype = Parent4.prototype`获得`Parent4`原型对象的内容，此过程中也减少了一次实例化代码调用。

缺点：Child4的原型对象引用了Parent4的原型对象，所以Child4的实例无法辨认是由Child4创建的还是Parent4创建的。原因是实例的`constructor`指向的是Parent4。

下面的代码可以优化这一点。

```js
// 组合继承优化2
function Parent5() {
  this.name = 'parent5'
  this.play = [1, 2, 3]
}
function Child5() {
  Parent5.call(this)
  this.type = 'child5'
}
Child5.prototype = Object.create(Parent5.prototype) //使用Object.create来使父子原型对象的隔离
Child5.prototype.constructor = Child5

let s7 = new Child5()
console.log(s7 instanceof Child5, s7 instanceof Parent5)
console.log(s7.constructor)
```

这种方式将`constructor`指向Child5。就不会出现分不清是谁创建的问题了。

# 本文参考

 - 《JavaScript高级程序设计》 6.2.3-原型模式
