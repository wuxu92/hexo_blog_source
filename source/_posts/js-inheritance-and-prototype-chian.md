---
title: 【翻译】JavaScript的继承与原型链
date: 2016-02-15 16:58:11
categories:
- programming
tags:
- js
- fe
- translate
---
这是 MDN上的[一篇文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)，介绍JavaScript的继承和原型链的基础知识，在此简单翻译一下，加深理解。当然MDN上已经有了[中文的版本](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)。

对于有基于类语言（class-based language, 如Java和C++）经验的开发者来说，初接触JavaScript可能有一些疑惑，因为它是动态类型的，并且本身不提供类型实现（class implementation），虽然ES6中引入了`class`关键字，但是也只是一个语法糖，JavaScript仍然是原型继承的（prototype-based）。

<!-- more -->
说到继承，JavaScript只有一种结构：对象，每个对象都有一个指向其原型对象的内部链接，原型对象又有自己的原型，以此类推直到对象的原型是`null`。根据定义，null没有原型，它作为原型链的最后一环。

原型继承常常被看作是JavaScript的一个弱点，实际上原型继承模型比传统的类型继承模型更加强大。举例来说，在原型继承的基础上构建一个类型模型并不困难，但是相反地要在类型系统上简历原型链模型却很困难。

## 基于原型链的继承
### 继承属性/inheriting properties
JavaScript对象是属性（这里指对象自己的属性）的动态“包”。JavaScript对象有一个指向原型对象的连接，当试图访问一个对象的属性时，不只是在对象本身寻找，而且会在它的原型对象，以及原型的原型上搜索属性，直到找到匹配的属性名，或者到原型链结束。

> 根据ECMAScript标准，符号 `someObject.[[Prototype]]`被用来指定 someObject的原型，这等价于JavaScript的 `__proto__`（现已弃用），从ECMAScript6开始， [[Prototype]] 可以使用访问器：[Object.getPrototypeOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) 和 [Object.setPrototypeOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 访问

下面的代码演示当访问一个对象的属性时发生的行为：


```js
// Let's assume we have object o, with its own properties a and b:
// {a: 1, b: 2}
var o = {a: 1, b: 2}
// o.[[Prototype]] has properties b and c:
// {b: 3, c: 4}
Object.setPrototypeOf(o, {b:3, c:4})
// Finally, o.[[Prototype]].[[Prototype]] is null.
Object.setPrototypeOf(Object.getPrototypeOf(o), null)

// This is the end of the prototype chain as null,
// by definition, null has no [[Prototype]].
// Thus, the full prototype chain looks like:
// {a:1, b:2} ---> {b:3, c:4} ---> null

console.log(o.a); // 1
// Is there an 'a' own property on o? Yes, and its value is 1.

console.log(o.b); // 2
// Is there a 'b' own property on o? Yes, and its value is 2.
// The prototype also has a 'b' property, but it's not visited. 
// This is called "property shadowing"

console.log(o.c); // 4
// Is there a 'c' own property on o? No, check its prototype.
// Is there a 'c' own property on o.[[Prototype]]? Yes, its value is 4.

console.log(o.d); // undefined
// Is there a 'd' own property on o? No, check its prototype.
// Is there a 'd' own property on o.[[Prototype]]? No, check its prototype.
// o.[[Prototype]].[[Prototype]] is null, stop searching,
// no property found, return undefined
```
设置一个对象的属性会生成一个自己的属性，获取或者设置属性的行为规则的唯一例外是当一个继承属性有[一个getter或者一个setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Defining_getters_and_setters)的时候。

### 继承方法/inheriting methods
JavaScript没有基于类的语言定义的那种形式的“方法/method”，在JavaScript中，任何函数都能以一个属性的方式添加到对象上，一个继承的函数的行为和其他的属性一样，与上面示例代码中的属性遮蔽（property shadowing）类似，常称为方法覆盖（method overriding）。

当执行一个继承的函数时，该函数中的 `this`会指向继承对象，**而不是原型对象**（这里的原型对象即拥有该方法的对象/where the function is an own property）。

```js
var o = {
  a: 2,
  m: function(b){
    return this.a + 1;
  }
};

console.log(o.m()); // 3
// When calling o.m in this case, 'this' refers to o

var p = Object.create(o);
// p is an object that inherits from o

p.a = 12; // creates an own property 'a' on p
console.log(p.m()); // 13
// when p.m is called, 'this' refers to p.
// So when p inherits the function m of o, 
// 'this.a' means p.a, the own property 'a' of p
```

## 创建对象并由此产生原型链的几种方法
### 使用普通语法创建对象

```js
var o = {a: 1};

// The newly created object o has Object.prototype as its [[Prototype]]
// o has no own property named 'hasOwnProperty'
// hasOwnProperty is an own property of Object.prototype. 
// So o inherits hasOwnProperty from Object.prototype
// Object.prototype has null as its prototype.
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// Arrays inherit from Array.prototype 
// (which has methods like indexOf, forEach, etc.)
// The prototype chain looks like:
// a ---> Array.prototype ---> Object.prototype ---> null

function f(){
  return 2;
}

// Functions inherit from Function.prototype 
// (which has methods like call, bind, etc.)
// f ---> Function.prototype ---> Object.prototype ---> null
```

### 使用构造器创建对象
在JavaScript中，构造器只是恰好被[ new 运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)调用的函数而已。

```js
function Graph() {
  this.vertices = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertices.push(v);
  }
};

var g = new Graph();
// g is an object with own properties 'vertices' and 'edges'.
// g.[[Prototype]] is the value of Graph.prototype when new Graph() is executed.
```

### 使用 Object.create
ECMAScript 5 引入了一个新方法： [Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) ，可以使用这个方法创建的对象，新对象的原型是函数的第一个参数。

```js
var a = {a: 1}; 
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (inherited)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); 
// undefined, because d doesn't inherit from Object.prototype
```

### 使用class关键字
ECMAScript 6 引入了一组关键字实现了 [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)，尽管对于熟悉基于类的语言的开发者来说，这些新的结构看起来很熟悉，但是它们（和基于类的语言）并不相同，**JavaScript仍然是基于原型的**，这些新的关键字包括： class, constructor, static, extends, and super.

```js
"use strict";

class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

var square = new Square(2);

square.sideLength = 3
```

## 性能
在原型链上查找属性花费更多时间，这对性能有副作用，对于性能要求很高的情况下很重要，特别是尝试访问一个不存在的属性总是会遍历整个原型链。

当迭代一个对象的所有属性时，原型链上的每一个可列举的（enumerable）属性都将是可列举的。

为了检查一个对象的一个属性是自己定义的而不是在原型链上的某处所定义的，需要调用`hasOwnProperty` 方法，所有对象都从 `Object.prototype`中继承了这个方法。当然向上面示例中，如果使用`var a = Object.create(null)` 创建对象则没有这个方法。

值得注意的是，`hasOwnProperty`方法是JavaScript中唯一一个只处理当前对象属性而不遍历原型链的方法。

Note: 要判断对象是否存在某个属性，不能简单地通过判断属性的值是否是undefined来决定，因为属性可能是存在的，而其值正好被设置为undefined

## 不好的实现：扩展本地原型
一个常见的错误做法是去扩展 `Object.prototype`或者扩展内置的原型。这种技术被称为“monkey patching”，它破坏了封装，但是有一些常见的框架使用了这种技术，比如 Prototype.js，即使这样也没有足够的理由去使用非标准方法破坏内置的类型系统。

我们扩展内置原型的唯一理由是引入新的JavaScript引擎特性，比如Array.forEach特性。

```js
function A(a){
  this.varA = a;
}

// What is the purpose of including varA in the prototype when A.prototype.varA will always be shadowed by
// this.varA, given the definition of function A above?
A.prototype = {
  varA : null,  // Shouldn't we strike varA from the prototype as doing nothing?
      // perhaps intended as an optimization to allocate space in hidden classes?
      // https://developers.google.com/speed/articles/optimizing-javascript#Initializing instance variables
      // would be valid if varA wasn't being initialized uniquely for each instance
  doSomething : function(){
    // ...
  }
};

function B(a, b){
  A.call(this, a);
  this.varB = b;
}
B.prototype = Object.create(A.prototype, {
  varB : {
    value: null, 
    enumerable: true, 
    configurable: true, 
    writable: true 
  },
  doSomething : { 
    value: function(){ // override
      A.prototype.doSomething.apply(this, arguments); // call super
      // ...
    },
    enumerable: true,
    configurable: true, 
    writable: true
  }
});
B.prototype.constructor = B;

var b = new B();
b.doSomething();
```
最重要的部分是：

- 类型被定义在 .prototype 中
- 使用 Object.create()来继承

## prototype和Object.getPrototypeOf
对于来自Java和C++的开发者来说，JavaScript可能有一点令人疑惑。它是全动态的，全运行时的，并且完全没有classes，所有的都是实例（对象），即使我们模拟出的“类”，也只是一个函数对象。

我们已经发现函数A有一个叫做 `protptype` 的特殊属性。这个特殊的属性用于JavaScript的 new 操作符。对prototype对象的引用被复制到新实例的内部 [[Prototype]] 属性。例如，当使用 `var a1 = new A()` JavaScript设置 `a1.[[Prototypr]] = A.prototype`。当你访问实例的属性时，JavaScript首先检查它们是否直接存在于该对象中，如果不是，它会在 [[Prototype]] 中查找。这意味着在 prototype 对象中定义的内容**会在所有实例中共享**，这意味着你可以之后修改原型，这种改变将影响所有现存的实例。

在上面的例子中，如果 `var a1 = new A(); var a2 = new A()`，然后 `a1.doSomething` 将会指向 `Object.getPrototypeOf(a1).doSomething`，这和 你定义的A.prototype.doSomething 是相同的，也就是 `Object.getPrototypeOf(a1).doSomething == Object.getPrototypeOf(a2).doSomething == A.prototype.doSomething`

简单地说，原型是类型，而 `Object.getPrototypeOf()`是对实例的。

[[Prototype]] 是递归的，就是 `a1.doSomething`, `Object.getPrototypeOf(a1).doSomething`, `Object.getPrototypeOf(Object.getPrototypeOf(a1)).doSomething` etc., 直到找到 doSomething 属性，或者 Object.getPrototypeOf 返回空。

所以，当调用：

```
var o = new Foo()
```
时，JavaScript实际执行：

```js
var o = new Object()
o.[[Prototype]] = Foo.prototype
Foo.call(o)
```
这样的流程，当执行下面的操作时：

```
o.someProp
```
js引擎首先检查o是否有 someProp 这个属性，如果没有，则检查 `Object.getPrototypeOf(o).someProp` 如果也没有，则继续检查 `Object.getPrototypeOf(Object.getPrototypeOf(o)).someProp` and so on.

## 结论
在用原型继承编写复杂代码前理解原型继承模型十分重要。同时，还要清楚代码中原型链的长度，并在必要时结束原型链，以避免可能存在的性能问题。此外，除非为了兼容新 JavaScript 特性，否则，永远不要扩展原生的对象原型。

Over