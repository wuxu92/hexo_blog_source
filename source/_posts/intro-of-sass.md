---
title: Sass介绍
date: 2016-04-04 15:38:41
categories:
- programming
tags:
- css
- Sass
- fe
---

前面几天一直忙着改论文，终于捯饬一篇出来投出去了，接下来可以有一段时间休息了。可以把这几天玩的东西整理整理了。这段时间主要玩的东西是Sass和Docker。其实这两样在以前都有一点点接触过，Sass和LESS大概在学CSS之后不久就有了解，只是觉得还要装一个编译器挺麻烦的，还不如直接写CSS；Docker也是大概大四的时候接触到，当时在服务器上装了一下，可是由于当时的网络实在不好，拉取镜像总是很慢或者失败，就没玩了。

这一篇主要记录一下Sass的简单认识。我们知道Sass是CSS的一个预处理器，也可以看作是一种更强大的CSS扩展编程语言，其官网 [http://Sass-lang.com/](http://Sass-lang.com/) 可以看出来Sass的目标是一种编程语言，让CSS的编写更加方便，更加像一门编程语言。其实还有一种也很常见的CSS预处理语言： [LESS](http://lesscss.org/)。LESS 也被广泛使用，LESS的特性是可以直接被浏览器使用，而LESS文件通常比编译出来的CSS文件小很多，这可以带来一定的性能优势，但是需要在客户端解析，这又导致一定的性能开销。关于LESS和Sass有很多比较，就不啰嗦了，反正我只玩了一下Sass，对LESS不甚了解。据说新版的Bootstrap会转向Sass，看来Sass是一个不错的选择。


## 安装和使用
这类预处理工具多由ruby开发，实在的，现在很多工具类应用都用ruby或者nodejs开发了，尤其是面向开发者使用工具。ruby和nodejs有很大的优势，倒是python见得少了些。要安装Sass编译器需要先安装ruby，然后用gem安装Sass。在Linux下面还是比较方便的，在Windows下就有些麻烦了，并且国内的网对gem库并不那么友好，常需要切到淘宝的镜像才能快速解决。
<!-- more -->
```
sudo gem install sass
```
现在有很多在线的编译工具，而且很多是免费的，使用也很方便，如果不需要实时查看结果的话，可以用支持Sass的编辑器先写好Sass然后用在线工具编译成css。我用的在线编译工具是: [http://www.Sassmeister.com/](http://www.Sassmeister.com/)

Sass文件就是普通的文本文件，可以使用CSS的语法，后缀为 `.scss`，意味 Sassy CSS。安装了本地Sass工具的话，可以直接：

```
sass boot.scss boot.css
```
生产CSS文件，如果不带css文件名则将结果显示在标准输出（屏幕）上。Sass还提供4中编译风格选项

- nested 嵌套缩进的css代码，默认使用这个值
- expanded 无缩进的扩展的css代码
- compact 简介格式的css代码
- compressed 压缩的css代码

生产环境中可以用 `sass --style compressed boot.scss boot.css` 生成css文件。

## 基本语法

### 变量
变量是这类css预处理器的重要功能之一，Sass使用以`$` 开头的变量形式，这对于习惯PHP风格的开发者来说非常友好，甚至Sass的用法和PHP有点类似：

```
$font-stack: Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  coloe: $primary-color;
}
```
这段代码很好理解，和css很像，不过是可以定义变量了，变量很明显是无类型的，也不需要双引号什么的括起来，一行定义一个就是了。编译时会将变量对应的值替换到body中的变量处，上面的Sass代码会编译为：

```
body {
  font: 100% Helvetica, sans-serif;
  color: #333;
}
```
还可以将变量嵌入到CSS选择器或CSS属性中：

```
$side: left;
.round {
  border-#{$side}-radius: 5px;
}

.col-#{$side} {
  ...
}
```
使用 `#{$var}`作为变量的占位符。这个在栅格布局中经常会结合for循环使用生成各个column的宽度，具体后面会介绍。

### 计算功能
Sass支持对变量进行计算，当然也支持直接使用常量表达式：

```
$right = 10px;
$i = 10;
body {
  margin: (14px/2);
  top: 50px + 100px;
  right: $right * 110%;
  left: $i * 10px;
}
```
Sass的计算直接使用带单位的数计算，也可以单位写在在后面，语法约束好像比较松散的。Sass支持的运算符包括： `+, -, *, /, %`。

### 嵌套（nesting）
写CSS时一个很不爽的地方就是不支持嵌套，必需一个个选择器列出来写属性，Sass支持类似于html层级结构的选择器嵌套方式。不过嵌套不应该过深，否则会很难维护，就像直接写CSS时也不建议将选择器写的太过复杂。

```
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```
甚至属性也可以嵌套，比如border-color属性嵌套为：

```
p {
  border: {
    color: red;
  }
}
```
注意属性的嵌套和选择器嵌套的区别，属性嵌套后面加上了冒号，而选择器没有。在嵌套代码块内使用 `&`引用父元素，比如：

```
a {
  color: black;
  &:hover {
    color: blue;
  }
}
```
会被编译为：

```
a {
  color: black;
}
a:hover {
  color: blue;
}
```

### 注释
Sass支持标准CSS注释： `/* */`，这类注释会在编译的文件中保留，但是在压缩模式中这类注释也可能被删除，可以在 `/*` 加上感叹号表示是**重要注释**，这样即使是压缩变异模式也会保留这行注释，如： `/*! Attention */`

Sass还支持单行注释 `//`，这种注释不出现在编译后的文件中。

## 代码重用
### 继承
Sass的选择器可以继承另一个选择器，使用 `@extend` 指令：

```
.class1 {
  border: 1px solid #eee;
}

.class2 {
  @extend .classs1;
  font-size: 120%;
}
```
这样class2就继承了class1的属性。不知道实际上有多少场景会使用到继承，不过这确实实现了复用，减少了修改的工作。关于继承编译的结果的理解，可以看官方文档的一个例子：

```
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.success {
  @extend .message;
  border-color: green;
}

.error {
  @extend .message;
  border-color: red;
}

.warning {
  @extend .message;
  border-color: yellow;
}
```
编译为：

```
.message, .success, .error, .warning {
  border: 1px solid #cccccc;
  padding: 10px;
  color: #333;
}

.success {
  border-color: green;
}

.error {
  border-color: red;
}

.warning {
  border-color: yellow;
}
```

### Mixin
Mixin有点像C语言的宏，是可以重用的代码块，使用 `@mixin` 定义一个代码块:

```
@mixin border-radius($radius: 4px) {
  -webkit-border-radius: $radius;
     -moz-border-radius: $radius;
      -ms-border-radius: $radius;
          border-radius: $radius;
}

.box { @include border-radius(10px); }
```
mixin强大在于可以指定参数，还可以指定参数的缺省值，然后使用 `@include`即可直接引入该代码块，上面的代码会编译为：

```
.box {
  -webkit-border-radius: 10px;
  -moz-border-radius: 10px;
  -ms-border-radius: 10px;
  border-radius: 10px;
}
```
mixin是Sass最为常用的功能，注意它和后面的函数的区别。

## 部分和import
Sass可以将一个CSS文件分成很多部分（partials），一般来说，这种包含部分CSS代码的文件使用下划线开头的命名，如： `_partial.scss`,对于使用下划线的文件，Sass编译器识别为partial file，而不会将它编译为CSS文件。

将文件分成小块可以看作是模块化的CSS，这可以提高CSS的可维护性。使用import指令在一个scss文件中包含其他的文件。常见的场景是将基础的、通用的属性定义在一个部分文件（partial file）中，然后在其他文件中引入：

```
// _reset.scss
html, body, ul, ol {
  margin: 0;
  padding: 0;
}
```

```
// base.scss
@import 'reset';

body {
  font: 100% Helvetica, sans-sarif;
  background-color: #efefef;
}
```
这里的 `@import reset`将会自动添加后缀搜索到文件，将内容整合到scss文件中。 `@import`指令还可以用来导入 css 文件，还可以指定路径导入文件，如： `@import "path/filename.css"`。

## 条件、循环和函数
### 条件
使用 `@if` 和 `@else`指令实现条件判断，例子：

```
p {
  @if 1 + 1 = 2 {
    border: 1px solide;
  } @else  {
    border: 2px dotted;
  }
}
```
### 循环
Sass支持`for` 、 `while`、 `each` 的循环：

```
@for $i from 1 to 10 {
　.border-#{$i} {
　　border: #{$i}px solid blue;
　}
}

// @while
$i: 6;
@while $i > 0 {
　.item-#{$i} { width: 2em * $i; }
　$i: $i - 2;
}

// @each
@each $member in a, b, c, d {
  .#{$member} {
　　background-image: url("/image/#{$member}.jpg");
　}
}

```

### 内置函数
Sass内置了一些颜色函数，以便生成系列颜色：

```
lighten(#cc3, 10%) 	// #d6d65c
darken(#cc3, 10%) 	// #a3a329
grayscale(#cc3) 	// #808080
complement(#cc3) 	// #33c
```

### 自定义函数
Sass可以自定义函数，可以像编写php，js函数那样方便的定义Sass的函数，函数可以用来进行一定的计算，允许有参数，有返回值，一个示例如下：

```
@function double($n) {
  @return $n * 2;
}

#sidebar {
  width: double(5px);
}
```
使用 `@function` 和 `@return` 指令定义函数名和返回值，其调用则直接通过函数名，不需要额外的前缀字符。在函数的定义要注意单位，对于乘运算如果参数带了单位，那么函数内部的乘法就不要带单位了。为了是函数的调用更明显，可以对自定义的函数进行文档说明：

```
// function double: return double size of parameter $n
// e.g. double(10px)
@function double($n)
  @return $n * 2;  // 不要 $n * 2px 这样会单位错误
}
```

## 一个示例
下面是一个简单栅格系统的css示例，这也是[某开始学习前端的同学](http://liuyanling.github.io/) 问我的一段Sass代码，因此我才去看了Sass：

```
* {
  margin: 0px;
  padding: 0px;
}

$gray: #1875e7;
$size_middle: md;
$size_large: lg;
$height: 50px;

@mixin position {
  position: relative;
}

@mixin Style {
  border: 1px solid $gray;
  float: left;
}

.container {
  @include Style;
}

.container * {
  @include Style;
  height: $height;
}

@mixin returns($size) {
  @for $i from 1 to 12 {
    .col-#{$size}-#{$i} {
      width: 100% / ($i * 12);
      @include position;
      margin: $i * 10px;
    }
  }
}

@media only screen and (min-width: 769px) {
  @include returns($size_large);
}

@media only screen and (max-width: 768px) {
  @include returns($size_middle);
}
```
写的很糟糕，尤其是命名什么的，不过可以用来理解Sass的结构。

Sass总体来说是值得学的，如果经常写css的话，Sass应该能带来很大的便利性，下一次写应该会尝试使用Sass而不是直接写CSS了。

参考：

- [http://Sass-lang.com/guide](http://Sass-lang.com/guide)
- [http://www.ruanyifeng.com/blog/2012/06/Sass.html](http://www.ruanyifeng.com/blog/2012/06/Sass.html)
- [http://Sass-lang.com/documentation/file.Sass_REFERENCE.html](http://Sass-lang.com/documentation/file.Sass_REFERENCE.html)
