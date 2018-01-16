---
date: 2015/07/27 12:34:50
title: 那些你可能不知道但是很有意思的CSS Tricks
categories:
- fe
tags:
- css
- fe
---

在去年的时候，Louis Lazaris在sitepoint上发了一篇文章 [http

参考：
[12 Little-Known CSS Facts](http://www.sitepoint.com/12-little-known-css-facts/)
[12 Little-Known CSS Facts (The Sequel)](http://www.sitepoint.com/12-little-known-css-facts-the-sequel/)

CSS 本身并不是非常复杂的语言，但是即使你写过很多的CSS样式，你总会感觉到对它并不了解，使用了很多年之后你还是会偶遇很多你没有用过的“有意思”的样式属性，本文作者列举了一些这样的Tricks。

## color属性不只是作用域文本 ##
color属性大家都很熟悉，一般用来设置文本(Text)的颜色属性，但是如果你把body标签的color设置为某种颜色，比如`color: yellow`几乎所有的东西就都会变成黄色的，包括图片的替代文本，列表的bullet，border等，但是`hr`标签并不会默认继承这个颜色，需要手动设置 `border-color: inherit`才会变成黄色。
color属性的spec中是这么定义的：


> This property describes the foreground color of an element’s text
content. In addition it is used to provide a potential indirect value
… for any other properties that accept color values.

## visibility属性可以设置为"collapse" ##
所有元素的visibility属性的默认值都是`visible`, 我们可以把它设置为`hidden`来隐藏一个元素，与`dispaly: none`表现不同的是，hidden的元素仍然会占据原来的空间（still occupy space）
visibility属性还有一个值就是`collapse`，它的表现一般和`hidden`是一样的，但是在table中表现会不一样，可能你已经想到了，在table元素中的行，行组，列，列组中使用`collapse`属性它的表现和`display:none`是一样的。
**不过用于浏览器对这个属性的表现并不一致，所以还是[不要使用](http://css-tricks.com/almanac/properties/v/visibility/)了**

## background属性有了新的内容 ##
在CSS 2.1中，background属性有color, image, repeat, attachment, position 5个“子内容”，在CSS3 中，添加很了一些新内容：

```css
background: [background-color] [background-image] [background-repeat]
            [background-attachment] [background-position] / [ background-size]
            [background-origin] [background-clip]
```
一个示例：

```
.example {
  background: aquamarine url(img.png)
	no-repeat
	scroll
	center center / 50%
	content-box content-box;
}
```
这些新属性在所有现代浏览器都能正常工作，只不过好像用处也不大。

## clip属性只在绝对定位元素上起作用 ##
clip属性可以用来对元素进行裁剪，它的值是一个图形，比如rect，或者是auto。
但是要注意，clip属性只对决定定位元素起作用，包括`position: absolute;`,和`position: fixed`两种情况。
使用:

```css
.example {
	position: absolute;
	clip: rect(110px, 160px, 170px, 60px;
}
```
它的表现可以参考这张图片: ![](http://cdn.impressivewebs.com/2012-05/clip-visual.jpg)
更多关于clip的内容，可以查看[这里](http://www.impressivewebs.com/css-clip-property/)

## 垂直百分比和容器的宽度有关而不是高度 ##
这个听起来有点奇怪，我们知道百分比计算宽度是基于容器的宽度的，但是实际上padding-top, margin-top, padding-bottom, margin-bottom这些属性的百分比属性也是基于容器的宽度的，而不是高度。
这是挺有意思的。

## border属性 is Kind of Like Inception ##
border属性我们会经常使用，比如这样子：

```css
.somecontainer {
	border: solid 1px black;
}
```
我们知道border属性是border-style, border=width, border-color三个属性的综合声明。但是其实这三个属性都可以对上下左右四条边分别设置属性：

```css
.somecontainer {
	border-width: 2px 5px 1px 0;
}
```
这样可以把border设置的很复杂，不过在实际使用中，并不要这么用，因为这样使得整个元素显得混乱而导致页面风格混乱。

## text-decoration 现在也成为了一种速记符（shorthand） ##
新的css规范中，可以这样使用text-decoration属性：

```css
a {
	text-decoration: overline aqua wavy;
}
```
第一个值是`text-decoration-line`, 第二个是`text-decoration-color`, 第三个是 `text-dcoration-style`属性
不过这个属性只在firefox中实现了。

## border-width属性可以接受关键字值 ##
现在border-width 可以接受 `medium`, `thin`, `thick`关键字了。他们在浏览器实现中大概是1px, 2px, 5px;不过在spec中并没有指定他们具体的值。

## 几乎没人使用botder-image ##
`border-image`看起来是个不错的属性，使某些表现变得整洁，有一个特别经典的例子是在一圈文字的border加一圈花or狗爪印？不过在现实中几乎没有人会使用这个属性，我也没有使用过。。

## empty-cells属性 ##
这个属性被所有主流浏览器支持，包括IE8。它的用法如下：

```css
table {
	empty-cell: hide;
}
```
这个属性用于table元素中空td的样式。或许会有用哦。 测试： [codepen](http://codepen.io/SitePoint/pen/yfhtq)


## font-style属性接受“oblique”值 ##
这个看起来和 `font-style: italic` 并没有什么区别，都是斜体字体，spec中是这么描述的：

> “…selects a font that is labeled as an oblique face, or an italic face if one is not.”

如果设置的字体没有真正的italic样式（face），那么`font-style:italic` 和 `font-style: oblique`的表现是一样的。

## word-wrap 属性和overflow-wrap属性是一样的 ##
`word-wrap`是微软提出的一个属性，所以在IE老版本中都有很好的支持，但是W3C决定使用`overflow-wrap` 替换掉 `word-wrap`。但是现在所有的浏览器都支持 `word-wrap`属性了，所以要替换也不是那么容易的事了。

没什么问题的话还是继续使用`word-wrap`吧。

## 待续 ##
