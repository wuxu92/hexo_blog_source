---
date: 2016/01/07 21:14:50
title: Vue.js的渲染机制与事件处理
categories:
- fe
tags:
- vue
- js
- fe
---
Vue.js真是一个好的东西！！！
## 条件渲染
在前面的例子中，已经使用过条件渲染指令 `v-if="var"` 了，还可以使用 `v-else` 实现更复杂的条件渲染，不过没有 `v-elif` 这样的指令了。

```html
<div id="render">
  <h1 v-if="isPass">Yes</h1>
  <h1 v-else>No</h1>
  <button v-on:click="passit">Change</button>
</div>

<script>
var render = new Vue({
  el: '#render',
  data: {
    isPass: true
  },
  methods: {
    passit: function () { this.isPass = !this.isPass }
  }
})
</script>
```
<!-- more -->

### template v-if
`v-if`是一个指令，需要将它添加到一个元素上，如果想要切换多个元素，需要使用一个 `<template>` 元素做包装，并在之上使用 `v-if`。 template是Vue.js提供的元素，它不会出现在渲染的结果中。

```
<template v-if="ok">
  <div>...</div>
  ...
</template>
```
还有一个 `v-show`指令用于根据条件展示元素，用法与 `v-if` 是一样的，它们的区别是由 `v-show`的元素会始终渲染并**保存在DOM中**，只是相当于设置css的 display属性，且不支持template元素，但是它可以和 `v-else` 结合使用。

熟悉jQuery的话，肯定对设置display很熟悉，设置display为none时，元素会隐藏起来，但是实际上它仍然在DOM树中，只是不显示而已，仍能通过选择器获得这个元素。而v-if 绑定的元素在条件不符合时节点是被删除的。它们有本质的不同。

`v-else`必须跟在`v-if`或者 `v-show`元素后面，否则它不能被识别。

一般来说，`v-if` 有更高的切换消耗而 `v-show` 有更高的初始渲染消耗。因此，如果需要频繁切换`v-show`较好，如果在运行时条件不大可能改变 `v-if` 较好。

## 列表渲染
写html时，列表元素是很麻烦的，因为大多是重复工作，而html又没有提供循环结构，所以要么在后台模板引擎（如php的smarty）用循环生成，要么用JavaScript生成，否则就只好手写了。

好在Vue.js提供了很方便的基于一个数组渲染一个列表的指令： `v-for`，其使用和脚本语言的`for in` 语法很像：

```html
<div id="comp">
  <p>{{message}}</p>
  <input type="text" v-model="newBook" v-on:keyup.enter="addBook">
  <ul>
    <li v-for="book in books">
      <span>{{book.title}}</span>
      <button v-on:click="removeBook($index)">X</button>
    </li>
  </ul>
  <button v-on:click="changeInputMessage">Change</button>
</div>
<script>
var comp = new Vue({
  el: '#comp',
  data: {
    message: "综合处理",
    books: [
      {title: "apue"},
      {title: "csapp"},
      {title: "c primer plus"}
    ]
  },
  methods: {
    addBook: function() {
      var book = this.newBook.trim();
      if (book) {
        this.books.push({title: book})
        this.newBook = ''
      }
    },
    removeBook: function(index) {
      this.books.splice(index, 1)
    },
    changeInputMessage: function() {
      this.message = this.message.slice(1) + this.message.slice(0, 1)
    }
  }
})
</script>
```
典型的结构是 `<ul><li v-for="iten in items"> ops on {% raw %}{{ item }}{% endraw %} </li></ul>`,注意循环渲染包括 `v-for`指令所在的元素，所以它定义在 li 节点上。

循环渲染作用域内有一个特殊变量 `{% raw %}{{$index}}{% endraw %}` ,它保存了当前元素的索引，这很有用，上面的例子就是用了这个变量。另外还可以像 `<div v-for="(idx, item) in items">` 这样为索引指定一个别名。

类似于`v-if`，也可以将 `v-for` 用在 `<template>`标签，以渲染一个包含多个元素的块，同时不渲染template元素本身。

## 数组变动自动检测
上面的例子中，在输入框输入内容回车后在 `books`中添加一个元素，相应渲染的列表也会添加，点击 X 按钮，会删掉数组中的一个元素，相应的页面的列表会自动删除，这是因为Vue.js包装了被观察数组的变异方法，这些操作数组的方法不是JavaScript原生的方法（即使原生有同名的方法），他们能触发视图的更新。这些方法是：

1. push()
2. pop()
3. shift()
4. unshift()
5. splice()
6. sort()
7. reverse()

这些方法都很好理解。

### 替换数组
变异方法，如名字所示，修改了原始数组。相比之下，也有非变异方法，如 `filter()`, `concat()` 和 `slice()`，不会修改原始数组而是返回一个新数组。在使用非变异方法时，可以直接用新数组替换旧数组：

```
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```
可能你觉得这将导致 Vue.js 弃用已有 DOM 并重新渲染整个列表——幸运的是并非如此。 Vue.js 实现了一些启发算法，以最大化复用 DOM 元素，因而用另一个数组替换数组是一个非常高效的操作

### track-by


### 不能追踪的变化
由于JavaScript的限制，下面的情况Vue.js不能检测到数组的变化

1. 直接使用索引设置元素： `vm.books[1] = {title: "Liux"}`;
2. 修改数组的长度，如 `vm.books.length = 0`; 直接设置length可能会导致数组被截断，Vue.js不能检测这种数据丢失

对于第一个问题，使用Vue.js提供的 `$set`方法设置值：

```
vm.books.$set(1, {title: "k&R C"});
```
Vue.js还提供了`$remove(item)`方法用于从目标数组中删除元素，remove内部使用的是 `splice`。

对于第二个问题只能避免使用设置length值，如果要置空，应该使用空数组替换变量。

### 在对象使用 v-for
熟悉js的话应该知道可以用for遍历object的属性，即遍历对象。 `v-for`指令也可以用来遍历对象属性： `<div v-for+"prop in object> ... </div>` 也可以 `<div v-for="(key, val) in object> ... </div>` 为键值提供一个别名。

### 指定循环次数
可以在`v-for`指令直接指定要循环的次数，即

```
<div>
  <span v-for="n in 10"> {{ n }} </span>
</div>
```
是不是很方便

## 方法处理器
之前已经使用过了，`v-on`绑定事件，事件定义在 Vue对象的 methods 域，事件定义带一个参数event，表示触发的时间，使用 `event.target` 获取出发事件的对象，这个原生的js事件是类似的，如果不需要event的信息，可以省略这个参数。

事件方法可以带参数，比如 `<button v-on:click="say('hi')">say hi</button>`，say函数定义可以有一个参数，如果要再显示传入event事件，可以在用: `say('hi', $event)`,这样在函数定义可以有两个参数（实际上可以不指定event参数而直接使用）。

### 事件修饰符
在处理表单的提交按钮点击事件，链接的点击事件时，经常要阻止原有的DOM事件，而只执行我们指定的事件处理函数，以前需要调用 `event.preventDefault()`（阻止默认行为）或者 `event.stopPropagation()`(防止冒泡，因为浏览器实现不同，这两个方法有不同的行为)。这两个函数有点麻烦，Vue.js提供了一个两个事件修饰符实现这个功能。

`.prevent` 和 `.stop` 两个修饰符对应上面两个函数的作用。

```html
<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat">
<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>
```
### 按键修饰符
Vue.js能方便的监听键盘事件，使用 `v-on:keyup.keycode=""` 的方式为按键绑定事件，其中的keycode是键盘按键代码，如 回车的keycode是13，但是记住keyCode有点麻烦，所以常用按键别名，常用按键别名如下：

- enter
- tab
- delete
- esc
- space
- up
- down
- left
- right

单字母按键也有别名。

今天先到这，下面是表单处理，过渡效果和组件

额外参考：

- [http://stackoverflow.com/questions/8312459/iterate-through-object-properties](http://stackoverflow.com/questions/8312459/iterate-through-object-properties)
