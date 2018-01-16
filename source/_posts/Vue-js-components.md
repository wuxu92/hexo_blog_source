---
title: Vue.js组件
date: 2016-01-10 14:01:53
categories:
- fe
tags:
- js
- fe
- vue
- mvvm
---
组件是Vue.js的重要部分。

## 基础
组件可以看作是自定义的完整模块，我们可以使用 `Vue.extend({...})` 创建一个组件构造器，该全局调用返回一个可复用的构造器。

extend 方法创建基础Vue构造器的子类，参数是一个对象，包含组件选项，这里要注意的特例是 `el` 和 `data` 选项，在 `Vue.extend()` 中，它们必须是函数。

要把这个构造器用作组件，需要用 `Vue.component(tag, constructor)` **注册**，这是一种全局注册方式，或者利用该构造器构建一个实例，然后用 `Vue.$mount()` 将该组件实例添加到DOM树上。一个示例如下：

```html
<div id='comp-ex-1"></div>
<div id="comp-ex">
  <Person></Person>
</div>
<script>
  var Person = Vue.extend({   // constructor
    template: "<div><span>name:</span> {{name}}  <span>Age: </span> {{age}}</div>",
    data: function() {
      return {
        name: 'li ming',
        age: 22
      }
    }
  });
  Vue.component('person', Person)  // regist component
  var you = new Vue({     // init component, render template
    el: '#comp-ex'
  })

  // 构造组件实例并挂接到DOM树
  var me = new Person({
    data: {
      name: 'wuxu',
      age: '23'
    }
  })
  me.$mount('#comp-ex-1');
</script>
```
<!-- more -->
### 局部注册
刚刚使用的 `Vue.commponent()` 全局函数做组件的全局注册，有时候我们不需要全局注册每个组件，可以让组件只能在其它组建内，用实例选项 `components` 注册，即在注册的对象参数中添加 components 成员，components成员的标签就只在改组建内使用，不在全局DOM树中使用局部注册的组件。类似上面的例子，取消Person的全局注册：

```html
<div id="comp-ex">
  <contact></contact>
</div>
<script>
  var Person = Vue.extend({   // constructor
    template: "<div><span>name:</span> {{name}}  <span>Age: </span> {{age}}</div>",
    data: function() {
      return {
        name: ' li ming',
        age: 22
      }
    }
  });
  var Contact = Vue.extend({
    template: "<span>Tel: </span> {{tel}}, <span>E-mail: </span> {{email}}<Person></Person><person></person>",
    date: function() {
      return {
        tel: '152-0101-1010',
        email: 'admin#163.com'
      }
    },
    components: {
      'person': Person		// 局部注册 person 标签
    }
  })
  Vue.component('contact', Contact)
  var you = new Vue({		// init component, render template
    el: '#comp-ex'
  })
</script>
```
### 语法糖
组件构造器构造，注册步骤有点繁琐，可以**直接传入选项对象**而不是构造器给 `Vue.component()` 和 `component` 选项。Vue.js在背后自动调用 `Vue.extend()`。

```
Vue.component('person', { template: "<div> template content </div>" })
```
这样讲构造、注册简化到一步了。

可以发现，上面的 data 域的值是一个函数，而不是一个对象，这是因为，如果使用一个数据对象，那么所有的组件实例都共享这一个对象，这基本不是我们想要的。

类似的， `el` 选项用在 `Vue.extend()` 中也需要是一个函数，不过一般不在构造函数中指定 el 选项。

### `is` 特性
一些 HTML 元素，如 <table>，限制什么元素可以放在它里面。自定义元素不在白名单上，将被放在元素的外面，因而渲染不正确。这时应当使用 is 特性，指示它是一个自定义元素。这个应该用的不多。

```html
<table>
  <tr is="my-component"></tr>
</table>
```
## Props 组件的数据
**组件实例的作用域是孤立的**，这意味着不能且不应该在子组件的模板内直接引用父组件的数据。可以使用`props`把数据传给子组件。props是组件数据的一个字段，期望从父组件传下来，**子组件需要显式地**用 `props` 选项声明，props选项的值是一个数组。

到现在，组件的构造选项对象的属性包括了： `template`，要渲染的内容，`data`，数据域，一般使用一个函数反悔对象，`props`，从父组件传递数据到子组件。

```js
Vue.component('child', {
  props: ['msg'],
  template: "<div><span>{{newMsg}}</span></div>"
})

// 渲染：
<child new-msg="hello"></child>
```
### 命名风格
HTML特性不区分大小写，名字形式为 camelCase 的prop用作特性时，**需要**转为 `kebab-case`(短横线隔开)。特意将上面的属性符合这个部分，在html中的属性使用短横线隔开，而在js的template中的标识使用的驼峰命名。

### props的动态
像上面的方式使用节点属性方式向子组件传递数据会失去动态性，也就是值不能是Vue实例的数据（data的成员），如果要绑定动态的值，应该使用 v-bind 指令，这样**每当父组件的数据变化时，也会传递给子组件**，使用 `v-bind`的缩写语法 `:`更加简单，如上面的template可以写为：

```html
<child :new-msg="msg"></child>
<!-- msg是构造选项对象data返回的对象的成员 -->
```
上面两种传递方式在传递数字时有一些不同，看下面的例子：

```html
<comp some-prop="1"></comp>
<comp :some-prop="1"></comp>
```
第一个数据传递的是字符串 `"1"`， 第二个则传递一个实际的JavaScript数字。虽然html渲染的时候并不太区分字符串和数字，但是注意有这种区别。


### props绑定类型
注意props绑定的数据默认是**单向绑定的**，当父组件的属性变化时，这种变化会传递给子组件，但是反过来却不会。这是为了防止子组件无意中修改了父组件的状态，不过这会让应用的数据流难以理解。不过可以使用 `.sync` 或 `.once` 绑定修饰符显式地强制双向或单次绑定。示例：

```html
<child :msg="hello"></child>		<!-- 默认单向绑定 -->
<child :msg.sync="hello"></child>	<!-- 双向绑定 -->
<child :msg.once="hello"></child>	<!-- 单次绑定 -->
```
双向绑定会把子组件的 `msg` 属性同步回父组件的属性。

不过要注意，如果绑定的值是一个对象或者数组，子组件内修改它会修改父组件的状态，这和后台语言的值引用与传递引用相似。

### props数据验证
在上面的示例中，props的值是一个数组类型，将要从父组件传递到子组件的变量分列出来。更加复杂的用法是使用一个对象，对每一个传递的值指定一些数据验证要求。这在多人合作，组件给其他人正确使用时很有用。下面是官方教程的一个例子：

```js
Vue.component('example', {
  props: {
    // 基础类型检测 （`null` 意思是任何类型都可以）
    propA: Number,
    // 必需且是字符串
    propB: {
      type: String,
      required: true
    },
    // 数字，有默认值
    propC: {
      type: Number,
      default: 100
    },
    // 对象/数组的默认值应当由一个函数返回
    propD: {
      type: Object,
      default: function () {
        return { msg: 'hello' }
      }
    },
    // 指定这个 prop 为双向绑定
    // 如果绑定类型不对将抛出一条警告
    propE: {
      twoWay: true
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    },
    // 转换函数（1.0.12 新增）
    // 在设置值之前转换值
    propG: {
      coerce: function (val) {
        return val + '' // 将值转换为字符串
      }
    },
    propH: {
      coerce: function (val) {
        return JSON.parse(val) // 将 JSON 字符串转换为对象
      }
    }
  }
})
```
可以看出，可以指定的验证方式有很多：`type`, `default`, `required`, `twoWay`, `validator`, `coerce`。
最常用的type指定数据的类型，可用的类型有：

1. String
2. Number
3. Boolean
4. Function
5. Object
6. Array
7. instanceof 检测

如果prop验证失败，Vue将拒绝在子组件上设置此值，如果使用的是开发版本会抛出一条警告。

## 组件间通信
子组件可以用 `this.$parent` 访问它的父组件，根实例的后代可以用 `this.$root` 访问它，父组件有一个**数组** `this.$children` 包含它所有的子元素。

尽管可以访问父链上任意的实例，不过子组件应当避免直接依赖父组件的数据，应当显式地使用props传递数据。另外子组件中修改父组件的状态是非常糟糕的，因为：

1. 这让父组件与子组件紧密地耦合
2. 这样的话，只看父组件很难理解父组件的状态，因为它可能被任意子组件修改！理想情况下，只有组件自己能修改它的状态

### 自定义事件
Vue实例实现了一个自定义事件接口，用于在组件树中通信。这个事件系统独立于原生DOM事件，做法也不同。每个Vue实例都是一个事件触发器。

- 使用 `$on()` 监听事件，
- 使用 `$emit()` 在它上面触发事件
- 使用 `$dispatch()` 派发事件，事件沿着父链冒泡
- 使用 `$broadcast()` 广播事件，事件向下传导给所有后代

> Vue事件不会自动冒泡，除非回调明确返回 true

自定义事件及分发的官方示例：

```html
<!-- 子组件模板 -->
<template id="child-template">
  <input v-model="msg">
  <button v-on:click="notify">Dispatch Event</button>
</template>

<!-- 父组件模板 -->
<div id="events-example">
  <p>Messages: {{ messages | json }}</p>
  <child></child>
</div>
```

```js
// 注册子组件
// 将当前消息派发出去
Vue.component('child', {
  template: '#child-template',	// template可以指向一个template的节点，这样很方便
  data: function () {
    return { msg: 'hello' }
  },
  methods: {
    notify: function () {
      if (this.msg.trim()) {
        this.$dispatch('child-msg', this.msg)
        this.msg = ''
      }
    }
  }
})

// 启动父组件
// 将收到消息时将事件推入一个数组
var parent = new Vue({
  el: '#events-example',
  data: {
    messages: []
  },
  // 在创建实例时 `events` 选项简单地调用 `$on`
  events: {
    'child-msg': function (msg) {		// 事件定义
      // 事件回调内的 `this` 自动绑定到注册它的实例上
      this.messages.push(msg)
    }
  }
})
```
另外可以使用 `v-on` 监听自定义事件；上面的例子的事件

```html
<child v-on:child-msg="handleIt"></child> <!-- 这是父组件中的代码 -->
```
当**子组件触发了 "child-msg" 事件，父组件的 handleIt 方法将被调用**。所有影响父组件状态的代码放到父组件的 handleIt 方法中；子组件只关注触发事件。

### 子组件索引
使用 `v-ref` 为子组件指定一个索引ID。指定索引与使用示例如下：

```html
<div id="parent">
  <user-profile v-ref:profile></user-profile>
</div>
<script>
  var parent = new Vue({el: '#parent'})
  var child = parent.$ref.profile	// 引用子组件
</script>
```

## Slot分发内容
为了让组件可以组合，我们需要一种方式来**混合父组件的内容与子组件自己的模版**，这个处理称为内容分发，也称为“嵌入包含”，可以看参考第二篇维基百科的内容。关于如何混合后面的例子可以清楚地理解。

Vue.js实现了一个内容分发API，它参照了当前的 Web组件规范草稿，使用特殊的 `<slot>` 元素作为原始内容的**插槽**。

### 编译作用域
先来看内容的编译作用域，一个原则就是： 

> 父组件模版的内容在父组件作用域内编译，子组件模版的内容在子组件作用于内编译

在使用组件时，一个常见的错误就是试图在父组件模版内将一个指令绑定到子组件上。下面是对之前的事件处理示例的简单修改：

```html
<template id="parent-template">
  <p>Message: {{ messages | json }}</p>
  <child v-show="childShow"></child> <!-- 在parent节点中定义 childShow -->
</template>
```
假定 `childShow`是子组件的属性，那么上面的例子**不能如预期工作**，即父组件不能控制子组件是否显示。父组件模版不知道子组件的状态。如果要绑定子组件内的指令到一个组件的根节点，**应当在子组件的模版内实现**：

```js
Vue.component("child-component", {
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      childShow: true	// 控制组件是否显示
    }
  }
})
```
### 使用slot
对于组件组合，父组件的内容将被**抛弃**，除非子组件模版包含了 `<slot>` 元素。如果只有一个没有特性的slot，整个内容将被插到它所在的地方，替换slot。

`slot`标签本身的内容被视为回退内容，回退内容在子组件的作用域内编译，只有当宿主元素为空并且没有内容插入时显示。如我们定义一个有一个 slot 的模版如下：

```html
<!-- 子组件模版，带一个slot -->
<template id="single-slot-comp">
  <div>
    <p>This is comp has one single slot</p>
    <slot>This is slot's default content to display</slot>
  </div>
</template>
<-- 父组件 -->
<div id="sslot-ex">
  <slot-comp-1>
    <span>parent content for slot</span>
    <div>战争与和平</div>
  </slot-comp-1>
</div>
<script>
  Vue.component('slot-comp-1', {
    template: '#single-slot-comp'
  })
  var sslot = new Vue({ el: '#sslot-ex'} )
</script>
```
### 命名slot
像上面一样只有一个未命名的slot，它就是默认slot，如果只有slot那么功能就显得有点单一了。Vue.js为`<slot>` 元素提供了一个特性 `name`，它用于配置如何分发内容。多个slot可以有不同的名字。命名 slot 将匹配有对应 slot 特性的内容片段。即在父组件模版内容的节点也加上 slot属性，用其指定分发到哪一个命名（同名）的子组件插槽（slot）中。官方示例就很好理解：

例如，假定我们有一个 multi-insertion 组件，它的模板为：

```
<div>
  <slot name="one"></slot>
  <slot></slot>
  <slot name="two"></slot>
</div>
```
父组件模板：

```
<multi-insertion>
  <p slot="one">One</p>
  <p slot="two">Two</p>
  <p>Default A</p>
</multi-insertion>
```
渲染结果为：
```
<div>
  <p slot="one">One</p>
  <p>Default A</p>
  <p slot="two">Two</p>
</div>
```
## 动态组件
多个组件可以使用同一个挂载点，然后动态地在它们之间切换，使用**保留的** `<component>` 元素，动态地绑到它的`is`特性。在之前的示例的基础上创建一个动态组件：

```html
<h3>Dynmic Component</h3>
<div id="dynamic-comp">
  <component :is="curVIew" keep-alive></component> <br>
  <button v-on:click="viewCh">change View</button>
</div>
<script>
  new Vue({
    el: '#dynamic-comp',
    data: {
      curVIew: 'slot-comp-1',
      views: ['slot-comp-1', 'child', 'contact'],
      viewIdx: 0
    },
    methods: {
      viewCh: function() {
        this.viewIdx = (this.viewIdx+1) % 3
        this.curVIew = this.views[this.viewIdx]
      }
    }
  })
</script>
```
上面 `component` 元素有一个 `keep-alive` 特性，用于将替换出去的组件的状态保留在内存中，保留状态可以避免重新渲染。

### 动态组件钩子
动态组件切换时，在切入组件前可能需要进行一些异步操作，为了控制组件切换时长，给**切入组件**添加 `activated` 钩子。`activate` 钩子只作用于动态组件切换或静态组件初始化渲染的过程中，不作用于使用实例方法手工插入的过程中。

还有一个`transition-mode`特性用于指定两个动态组件之间如何过度。默认情况下，进入和离开平滑地过度，使用这个特性可以指定另外另种模式： `in-out`, `out-in`，实现自定义的过渡效果。过渡效果使用 CSS实现。

```html
<component
  :is="view"
  transition="fade"
  transition-mode="out-in">
</component>
```

```css
.fade-transition {
  transition: opacity .3s ease;
}
.fade-enter, .fade-leave {
  opacity: 0;
}
```
这个例子在[官方文档](http://cn.vuejs.org/guide/components.html#transition-mode)可以查看

## 杂项

### 组件和v-for

### 可复用组件

### 异步组件

### 命名约定

### 递归组件

### 片段实例

### 内联模板

本文的示例代码：
[Vue.js Practice](http://demo.wuxu92.com/vue/index.html)

参考：

- [https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md)
- [http://zh.wikipedia.org/zh/Wikipedia:%E5%B5%8C%E5%85%A5%E5%8C%85%E5%90%AB](http://zh.wikipedia.org/zh/Wikipedia:%E5%B5%8C%E5%85%A5%E5%8C%85%E5%90%AB)