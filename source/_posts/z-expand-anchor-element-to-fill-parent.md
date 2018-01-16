---
date: 2015/07/24 12:34:50
title: A标签(Anchor Tag)占满父元素的实现
categories:
- fe
tags:
- css
- fe
---

A标签默认是inline元素，所以标签内容的长短决定了鼠标hover的范围，在有些时候，比如某些导航列表中，需要hover父标签（li）时能hover到a标签。

在制作一个API文档的侧边栏导航列表的时候，html大概是这样的

```
<div class="toc-macro absolute">
<ul class="toc-indentation">
  <li><a href="#id-1">1. 绑定手机号</a></li>
  <li><a href="#id-2">2. 查询手机号码是否可以绑定</a></li>
  <li><a href="#id-3">3. 添加邮箱</a></li>
  <li><a href="#id-4">4. 检查邮箱是否可以添加</a></li>
</ul>
</div>
```


其他的样式就不具体写了，其实现的效果是在右侧边栏实现一个列表导航。一些主要的样式大概如下：

```css
.toc-macro{ position: fixed; top: 100px; }
.toc-macro ul {list-style-type:none; }
.toc-macro a { text-decoration: none; }
.toc-macro li { padding: 6px 8px 2px;text-decoration:none; }
li:hover { background-color: #efefef;color: #7DB75C; }
.toc-macro a:hover { background-color: #efefef;color: #7DB75C; }
```

其中使用了决定定位时导航不会随着滚动条而滚动，使我们在查看API文档时总是能看到这个导航。
这样子基本符合我们的需求了，但是有一个小小的问题，那就是鼠标放到导航上如果没有放到a链接上面，li标签会变色，但是由于鼠标没有在a上所以还是箭头的形状。。。这个小小的瑕疵需要解决一下。

首先我们想到的是把a标签的宽度设置到 100%：

```
.toc-macro a { display:inline-block; width: 100%; }
```

但是这样似乎并没有按照我们所想的表现出来。我们需要再加一些东西，让a标签的inline-block可以把宽度填充

```
.toc-macro li { vertical-align: middle; line-height: 15px;}
```

另外还有一种实现思路那就是把div下所有标签的display属性设置为block,同时a的height设置为100%：

```
.toc-macro ul { display: block; }
.toc-macro li { display: block;}
.toc-macro a { display:block; height:100%; }
```

这样实现应该也是可行的，不过没有测试过了。
