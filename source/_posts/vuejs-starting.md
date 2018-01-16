---
date: 2016/01/05 12:34:50
title: 开始学习 vue.js
categories:
- fe
tags:
- vue
- js
- fe
---

2015年，Vue.js在国内的前端圈子内可是大红大紫了一把，作为国内开发者（尤雨溪）开源的作品，在github上获得众多star与 fork，也成为laravel官方推荐的js框架，大有可取代jQuery的势头。目前前端JS框架比较流行的，jQuery不用说是使用最广泛的，现在随便浏览一个网站都基本都使用了它，jQuery在Web技术发展中可是立下了汗马功劳了，不过jQuery因为时代原因，也越来越被开发者所“嫌弃”，认为jQuery只能算一个类库，而算不上现代的js框架。即使这样，也不可否认,jQuery是非常优秀的。

现在jQuery之外有了更多选择（这些选择不再是ext，ember这些了），那就是 Angular， react等等，虽然它们所要实现的功能并不相同，但是也经常拿他们一起比较。可以参考[这个讨论](https://www.zhihu.com/question/38989845)。

## 开头
<!-- more -->

今天中午开始看了一会 Vue.js 的示例，简单使用了一下，其功能真是非常惊艳，相见恨晚，其数据操作和jQuery完全是不同的思路，就是非常简单直接，使js的开发更加有feel了。不过玩了一会后，手机被我搞坏了，充不了电，开不了机，下午就跑去数码大厦修手机了，修完手机回来刷机一直有问题，搞到好晚才刷到最新的CM，不提。
<!-- more -->
开发时应该使用[开发版本](http://vuejs.org/js/vue.js)，可以有友好的错误提示，部署时使用[生产版本](http://vuejs.org/js/vue.min.js)，体积更小。使用时，直接使用 script 标签引入即可。

Vue.js官方的文档非常棒，毕竟是中国人开发维护的项目，中文文档看起来很好理解，<del>现在作者在国内做了一个文档镜像</del>：[http://vuejs.org/guide/instance.html](http://vuejs.org/guide/instance.html)，访问速度快很多了，查文档推荐去这里。

官方文档的起步、概述讲的很详细，就不赘述了，跟着走一遍就好了，这部分是基础，也很重要，对js有些经验的话，应该很快就能过完。

> Vue.js 的核心是一个响应的数据绑定系统，它让数据与 DOM 保持同步非常简单。在使用 jQuery 手工操作 DOM 时，我们的代码常常是命令式的、重复的与易错的。Vue.js 拥抱数据驱动的视图概念。通俗地讲，它意味着我们在普通 HTML 模板中使用特殊的语法将 DOM “绑定”到底层数据。一旦创建了绑定，DOM 将与数据保持同步。每当修改了数据，DOM 便相应地更新。这样我们应用中的逻辑就几乎都是直接修改数据了，不必与 DOM 更新搅在一起。这让我们的代码更容易撰写、理解与维护。

## Vue实例
一个Vue应用的起步都是使用全局构造函数`Vue` 创建一个 **Vue的根实例**。该函数的参数是一个js对象（键值对，选项对象），一般的指定键为 el, data, methods;el是一个css选择器，代表这个Vue实例对应的dom节点，比如 '#app'; data是这个实例持有的数据，它是一个js对象，其成员的键绑定DOM节点中同名的符号，methods是方法的集合，也是一个js对象，其键是方法名，值为一个函数。

一个Vue实例其实正是一个 MVVM模式中的ViewModel，可以将一个实例保存为一个本地变量，用于对该节点的引用。

每一个Vue实例都会**代理**其data对象里所有的属性（可以直接用示例变量名引用data中的键，而不需要通过data域）。注意，只有被代理的属性是**响应**的，在实例创建**之后**添加到实例的属性不会触发视图的更新。除了数据属性，Vue实例还暴露一些有用的实例属性与方法。这些属性与方法都使用前缀 `$` 以区分于代理的数据属性。比如data域使用 `vm.$data`，还有 `vm.$el`、`vm.$watch`等，下面是官方的示例：

``` js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true

// $watch 是一个实例方法
vm.$watch('a', function (newVal, oldVal) {
  // 这个回调将在 `vm.a`  改变后调用
})
```

### Vue实例的生命周期
Vue实例的创建有系列初始化步骤，如建立数据观察，编译模板，创建必要的数据绑定。在此过程中，它会调用一些**生命周期钩子**，这些钩子会在对应的阶段自动调用，Vue实例的生命周期图示如下：
![](/images/lifecycle.png)
其中红色圆角方框中的就是暴露的钩子。

## 基础语法
Vue可以看作一个模板引擎，有自己的数据绑定语法，这里主要就介绍数据绑定。

Vue 的模板是**基于DOM**实现的，这意味着所有的Vue模板都是可解析的有效的HTML，且通过一些特性做了增强。**Vue 模板从根本上不同于基于字符串的模板**，作者让我们牢记这一点。

### 插值
数据绑定最基础的形式是文本插值，使用 "Mustache"(双大括号)语法。在HTML中任意地方（Vue实例指定的DOM节点）插入 Mustache 标签就会被相应数据对象的属性的值替换，**每当这个属性变化时，显示的值也会自动更新**。也可以只处理单次插值，后面的数据变化就不会再引起插值更新，其语法是在大括号内的第一个字符插入一个星号和一个空格。

```html
<span> {{* msg }} <span>
```

对于html内容，使用三 Mustache标签,将数据解析为html而不是纯文本，双大括号则直接输出纯文本。（注意渲染不受控制的HTML内容是非常危险的，可能导致XSS攻击）。

Mustache 标签可以用在HTML特性（Attributes）内。

### 绑定表达式
放在大括号标签内的文本称为**绑定表达式**，在Vue.js中，一段绑定表达式由一个简单的JavaScript表达式和一个可选的一个或多个**过滤器**构成。

注意一个绑定只能包含**单个**的 **表达式**，而**不能是语句**或者流程控制，三元操作符，函数调用是可以的，赋值操作和if等是不允许的。

Vue.js允许在表达式后添加可选的**过滤器（filter）**，以管道符(|)指示。如 
```
{{ message | capitalize }}
```
这里的capitalize即是所谓“过滤器”，这个过滤器其实只是一个JavaScript函数，返回大写化的值，Vue.js提供数个内置过滤器，也可以自己开发过滤器。


过滤器函数可以指定多个参数，它始终以表达式的值作为第一个参数，后面指定后续的参数，如：

```
{{ message | filter 'arg1' arg2 }}
```
这里 arg1 字符串作为filter函数的第二个参数，arg2可以是一个表达式，将其计算的结果作为第三个参数。


管道(|)不是JavaScript的语法，所以不要在表达式内使用过滤器，只能添加到表达式的后面，Vue.js的过滤器可以串联，就像Linux中的管道一样。

### 指令
指令（directive）是特殊的带有前缀`v-`的特性，指令的值限定为**绑定表达式**，指令的职责就是当其表达式的值改变时把偶写特殊的行为应用到DOM上，常见的指令：`v-if`, `v-for`, `v-bind`, `v-on`。几个示例：

```html
<div id="bind">
  <a href="" v-bind:href="url" v-bind:target="target">{{url}}</a>
  <br>
  <button href="" v-on:click="update">{{text}}</button>
  <br>
  <a href="" v-if="greeting" v-bind:target="target">{{greet}}</a>
  <br>
</div>
```

上面的 `v-bind:href` 在冒号后面的 href称为**参数**，v-bind可以用于绑定节点的属性值。还有一个**修饰符**，它以半角句号开始的特殊**后缀**，用于表示指令应当以特殊方式绑定，例如常用的 `.literal` 修饰符表示将值解析为一个字面量字符串而不是一个表达式，常用的在事件绑定中使用修饰符指定对应的按键，例如为input绑定回车事件： `v-on:keyup.enter="addBook"`。

```js
var bind = new Vue({
  el: '#bind',
  data: {
    target: '_blank',
    url: "http://blog.wuxu92.com",
    text: 'update',
    greet: "hello",
    greeting: true
  },
  methods: {
    update: function() {
      this.greeting = !this.greeting
    }
  }
})
```
这是针对上面的DOM节点的Vue实例。

### 缩写
使用 `v-` 前缀是标识模板中特定的Vue特性的视觉暗示。如果觉得使用前缀比较罗嗦麻烦，而且在构建单页面应用时，Vue.js会管理所有的模板，此时前缀就没有那么重要了，Vue.js为两个最常用的指令 `v-bind` 和 `v-on` 提供了特别的缩写。

v-bind 可以直接省略，而已 `:` 开后，因为bind指令肯定需要指定一个参数的，可以省略v-bind前缀。

v-on: 可以省略为 `@` 符号，如 

```
<a v-on:click="clickHandler">link</a>
```
等价于： 

```
<a @lick="clickHandler">link</a>
```

## 计算属性
Vue将数据绑定限制为一个表达式，如果需要多于一个表达式的逻辑，应该使用计算属性。计算属性是 Vue实例中的一个域 `computed`,将一个函数作为属性的值（这个函数必须有返回值），然后可以将计算属性绑定到标识上，等价于函数的计算值绑定,可以向绑定普通属性一样绑定计算属性，计算属性也是即时更新的。

```html
<div id="comput">
  a={{a}} and b={{b}}
  <br>
  <button @click="addOne">a+1</button>
</div>
<script> <!-- 这里的\是为了显式正常加的 -->
var computed = new Vue({
  el: '#comput',
  data: {
    a: 100
  },
  computed: {
    b: function() {
      return this.a * 2  // this.a can't omit this
    }
  },
  methods: {
    addOne: function() {
      this.a += 1
    }
  }
})
</script>
```
Vue 还提供了一个暴露的 $watch 方法，用于观察Vue实力上的数据变动，当一些数据需要根据其他数据变动时，watch很有用，但是实际上它的作用和计算属性是一样的，并且计算属性更加优雅且好理解，所以**更好的选择是使用计算属性**而不是调用 $watch 方法。

计算属性默认只提供 getter方法，就像上面的例子一样，当然我们也可以提供一个setter方法，下面是一个示例：

```js
computed: {
  b: {
    get: function() {
      return  this.a * 2
    },
    set: function(newVlaue) {
      ...	// do something here
    }
  }
}
```
注意，setter方法并不会破坏计算属性的get方法，也就是对应的标签显示的值只会是get方法计算的值。

更多细节参考： [http://cn-stage.vuejs.org/guide/reactivity.html#计算属性的秘密](http://cn-stage.vuejs.org/guide/reactivity.html#计算属性的秘密)

## Class与Style绑定
数据绑定还可以操作元素的class列表和它的内联样式。因为它们都是属性，我们可以用 `v-bind`处理它们，即计算出表达式的最终字符串后绑定上去，不过字符串拼接很麻烦又容易出错，所以Vue.js增强了Class和Style属性的绑定，这类数据绑定的表达式结果类型除了字符串外，还可以**对象或者数组**。如果对css样式有了解的话，会发现这样的处理会变得非常方便。

尽管可以用标签绑定class，但是不推荐`class="{% raw %}{{ className }} {% endraw %}"` 和 `v-bind:class` 混用，两者只选其一。

绑定Class和Style有两种方式，只需要掌握绑定数据里的一个对象这种方法即可，更详细的方法参考官方手册： [http://cn-stage.vuejs.org/guide/class-and-style.html](http://cn-stage.vuejs.org/guide/class-and-style.html)

### 示例
绑定class和style的示例如下：

```html
<div id="classbind">
  <div class="static" v-bind:class="bookClass">books class</div>
  <div class="static" v-bind:style="bookStyle">books style</div>
</div>
```
js

```js
var classbind = new Vue({
  el: '#classbind',
  data: {
    bookClass: {
      'class_a': true,    // class_a, class_b 是一个css类选择器名
      'class_b': false
    },
    bookStyle: {
      color: "red",
      'font-size': "22px"    // font-size做键需要引号引起来
    }
  }
})
```

> 注意，带有特殊字符（如上面的-）的字符串做JavaScript对象的键时，需要使用引号，引用其值时不能用点号，而要使用数组形式。

上面的节点可以像`classbind.bookClass.class_a = false` 这样操作删除相应节点的class_a。

style的绑定还可以是一个数组，将多个样式对象应用到一个元素上，如： `v-bind:style="[styleObjA, styleObjB]"`

### 自动前缀
当 `v-bind:style` 使用需要厂商前缀的 CSS 属性时，如 `transform`，Vue.js 会自动侦测并添加相应的前缀。 

完
