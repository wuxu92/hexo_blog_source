---
title: Vue.js的表单处理和过渡效果
date: 2016-01-08 10:53:12
categories:
- fe
tags:
- js
- fe
- vue
---
这部分主要是表单处理，动画和组件。
## 表单处理
在Vue.js的起步教程中就使用过 `v-model` 指令在表单控件元素上创建**双向数据绑定**。根据控件类型它自动选取正确的方法更新元素， `v-model`不过就是语法糖，在用户输入事件中更新数据，以及特别处理一些极端例子。

这里只要注意数据的双向绑定即可，如果输入框绑定的变量在其他地方也使用的话，修改输入框的值，其他的节点的值也是同步更新的。 如：

```
<span>Message is: {{ message }}</span>
<br>
<input type="text" v-model="message" placeholder="edit me">
```
这里每修改一次input的输入框内容，span显示的内容和input的值是同步的。

<!-- more -->
### checkbox
对于单个checkbox绑定的model就是checked or not,就比较简单： `<input type="checkbox" id="checkbox" v-model="checked">` checked是Vue实例中的数据域成员，可以操作这个值改变checkbox的状态。

对于多选框，需要绑定到一个数组上，总体来说还是很方便的：

```html
<div id="formbind">
  <input type="checkbox" v-model="checkedNames" value="Mike"> Mike <br>
  <input type="checkbox" v-model="checkedNames" value="John"> John <br>
  <input type="checkbox" v-model="checkedNames" value="Bob"> Bob <br>
  <span>checked: {{checkedNames | json}}</span>
</div>
<script>
var formbind = new Vue({
    el: '#formbind',
    data: {
      checkedNames: []
    }
})
</script>
```
### radio 与 select
接上面的例子：

```
<input type="radio" value="one" v-model="picked">one</input> <br>
<input type="radio" value="two" v-model="picked">two</input> <br>
radio result: {{picked}}
```
picked 标签的值等于选中的radio的value值，如果在Vue实例的构造函数中设置 `picked: "one"` 则渲染页面时第一个按钮会被选中。

与checkbox类似，对于单选的下拉框， 将select节点的 v-model 绑定为一个变量，对于多选的下拉框 `v-model`绑定为一个数组。如：

```html
<select v-model="selected">
  <option selected>A</option>
  ....
</select>

<select v-model="selecteds" mmultiple>
  <option>A</option>
  ...
</select>
Selecteds: {{ selecteds | json }}
```

### 值绑定
还可以结合v-for渲染下拉框的 option 节点，将下拉框的选项编码在 js 代码中，更加方便管理。

对于radio、select的option、checkbox节点的value一般直接硬编码为字符串，也可以使用Vue.js的`v-bind`指令将其绑定到一个Vue实例的动态属性上。

```html
<!-- 选中时，toggle的值为vue.a，取消选中时其值为 vue.b -->
<input type="checkbox" v-model="toggle" v-bind:true-value="a" v-bind:false-value="b">
<input type="radio" v-model="picked" v-bind:value="a">
<select v-model="selected">
  <!-- 对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
<script>
  typeof vm.selected // -> 'object'
  vm.selected.number // -> 123
<script>
```
### 参数特性
默认情况下，v-model 绑定的变量在input事件（即任何修改时）同步输入框的值，可以添加一个 `lazy` 特性，修改数据同步到 `change` 事件（即input失去焦点且内容变化时）。

还可以设置一个 `number` 特殊属性，将用户的输入自动保存为数字。

有时候希望input修改时延时同步输入框的值与数据，例如对用户名的检测，要在用户输入发生时发送Ajax请求到后台检查，但是如果每次修改都立即同步发送请求的话经常会导致卡顿，这时可以添加一个 `debounce` 特性做延时同步，即设置一个最小的延时，让同步在修改之后一段时间再发生。其用法如下：

```
{{ msg }}
<input v-mode="msg" debounce="500">
```
500即表示500毫秒后再同步msg的值。注意debounce只是延迟数据同步，不会延迟 input 事件的响应，若要延迟DOM事件，可以使用 **debounce过渡器**。

这里介绍了三个特性：

1. lazy
2. number
3. debounce

在Ajax请求与变量值关联时，常设置一个debounce特性。

## 过渡效果
使用jQuery可以很方便的添加元素的显式隐藏动画，加上一些js库之后可以做出非常复杂经验的动态效果，Vue.js也提供过度系统，可以在元素从DOM中插入或移除时自动应用过渡效果，Vue.js会在适当的时机触发CSS过度或动画，还提供了JavaScript钩子函数在过度过程中执行自定义的DOM操作。

应用过渡效果，需要在目标元素上使用 `transition` 特性，结合 `v-if`, `v-show`, `v-for` 等指令在节点变化时添加过渡效果，结合这些指令使用时只能添加元素插入和隐藏（删除）时的效果。

```html
<div v-if="show" transition="transIns"><h1>transition</h1><div>
```
当插入或者删除带过渡的元素时，Vue将执行下面的逻辑：

1. 尝试以 ID "transIns" 查找 JavaScript 过渡钩子对象——通过 `Vue.transition(id, hooks)` 或 `transitions` 选项注册。如果找到了，将在过渡的不同阶段调用相应的钩子。
2. 自动嗅探目标元素是否有 CSS 过渡或动画，并在合适时添加/删除 CSS 类名。
3. 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作（插入/删除）在下一帧中立即执行。

### CSS过渡
CSS过渡即使用CSS样式定义过渡效果，一个典型的CSS过渡是这样的：

```html
<div v-if="show" transition="expand">hello</div>
```
然后为其添加CSS样式: `.id-transition`, `.id-enter`, `.id-leave`，如果transition没有指定值，即id为空，默认使用类名： `.v-transition`, `v-enter`,`.v-leave`。添加CSS样式示例：

```css
.expand-transition {
  transition: all .3s ease;
  height: 30px;
  padding: 10px;
  background-color: #eee;
  overflow: hidden;
}

/* .expand-enter 定义进入的开始状态 */
/* .expand-leave 定义离开的结束状态 */
.expand-enter, .expand-leave {
  height: 0;
  padding: 0 10px;
  opacity: 0;
}
```
此外还可以提供JavaScript的钩子，通过 `Vue.transition("id", {})` 提供。如：

```js
Vue.transition('expand', {
  beforeEndter: function(el) {},
  enter: function(el) {..},
  afterEnter: function(el) {},
  enterCancelled: function(el) {},
  beforeLeave: function() {el},
  leave: function(el) {},
  afterLeave: function(el) {},
  leaveCancelled: function(el) {}
})
```
可以选择实现其中的一个，多个或所有方法。过渡效果的流程详解参考： [http://cn.vuejs.org/guide/transitions.html#u8FC7_u6E21_u6D41_u7A0B_u8BE6_u89E3](http://cn.vuejs.org/guide/transitions.html#u8FC7_u6E21_u6D41_u7A0B_u8BE6_u89E3)

注意，当有多个元素一起过渡时，Vue.js会批量处理，只强制一次布局。

## CSS动画
CSS动画的用法和CSS过渡相似，区别是在动画中 `v-enter` 类名在节点插入DOM后不会立即删除，而是在 `animationend` 事件触发时删除。

官方的示例如下：

```html
<span v-show="show" transition="bounce"> Look at me!</span>
```

```css
.bounce-enter {
  animation: bounce-in .5s;
}
.bounce-leave {
  animation: bounce-out .5s;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
@keyframes bounce-out {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(0);
  }
}
```
通过 `@keyframes` 定义关键帧实现动画效果。

## JavaScript过渡
可以只适用JavaScript钩子，不定义任何CSS规则，当只使用JavaScript过渡时， `enter`和`leave`钩子需要调用`done`回调，否则它们将被同步调用，过渡将立即结束。

通常为JavaScript过渡显示声明 `css: false`，这样Vue.js将跳过CSS检测，这样会阻止无意间让CSS规则干扰过渡。

具体使用参考官方示例： [http://cn.vuejs.org/guide/transitions.html#JavaScript__u8FC7_u6E21](http://cn.vuejs.org/guide/transitions.html#JavaScript__u8FC7_u6E21)

### 渐进过渡
transition 与 v-for 一起用时可以创建渐近过渡。给过渡元素添加一个特性 `stagger`, `enter-stagger` 或 `leave-stagger`。

因为过渡效果比较麻烦，并且也不是学习的重点，这里就跳多了，需要的直接查看官方文档吧。

这部分已经听多了，组件放到下一篇吧。

完