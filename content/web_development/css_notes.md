---
title: "CSS notes"
date: 2019-11-20 23:01
tag: web development
---

[TOC]

---

## CSS 介绍

CSS = 层叠样式表

Cascading Style Sheets

HTML表达结构，CSS表达样式

样式和内容/结构是分离的

CSS 在 HTML 中的使用有三种方法：

- 行内样式表（style属性）
- 内部样式表（style元素），放在头部
- 外部样式表（引用一个样式表文件）

### 行内样式表

```css
<!DOCTYPE html>
<html>
     <head>
          <title>例子</title>
          <meta charset = utf-8>
     </head>
     <body style="background-color: #FF0000;">
          <p>这个页面是红色的</p>
     </body>
</html>
```

### 内部样式表

```css
<!DOCTYPE html>
<html>
     <head>
          <title>例子</title>
          <meta charset = utf-8>
          <style type="text/css">
               body {background-color: #FF0000;}
          </style>
     </head>
     <body>
          <p>这个页面是红色的</p>
     </body>
</html>
```

### 外部样式表

外部样式表就是一个扩展名为css的文本文件。跟其他文件一样，你可以把样式表文件放在Web服务器上或者本地硬盘上。

例如，比方说你的样式表文件名为style.css，它通常被存放于名为style的目录中。

```css
<html>
     <head>
          <title>我的文档</title>
          <link rel="stylesheet" type="text/css" href="style/style.css" />
     </head>
     <body>
     ...
```

这种方法的优越之处在于：**多个HTML文档可以同时引用一个样式表**。换句话说，可以用一个CSS文件来控制多个HTML文档的布局。

## 基本的CSS语法

比方说，我们要用红色作为网页的背景色：

用HTML的话，我们可以这样：

```html
     <body bgcolor="#FF0000">
```

用CSS的话，我们可以这样获得同样的效果：

```css
     body {background-color: #FF0000;}
```

你会注意到，HTML和CSS的代码颇有几分相似。

---

## 背景样式

可以是整个页面，某个段落，或者某一个小片段

```html
<!DOCTYPE html>
<html>
     <head>
          <title>例子</title>
          <meta charset = utf-8>
         
     </head>
     <body style="background-color:rgba(0,255,0,1);">
          <p style="background-color:rgba(255,0,0,0.5);">
          另一个段落
          </p>
          <p>
          我的第一个HTML页面
          </p>
     </body>
</html>
```

背景效果：a. 颜色, b. 图像

颜色可以用英文和十六进制两种表示方式 `<p body style="background-color:#ff0000">`；

图片设置为背景： `<style="background-image:url(XXX.jpg)">`。

若图片尺寸较小，不能占满整个页面可设置重复属性，比如下面的几种方式：

不重复：`<style="background-image:url(XXX.jpg)；background-repeat:no-repeat>"`

只横向、纵向重复：`<style="background-image:url(XXX.jpg)；background-repeat:repeat-x/y>"`

设置图片位置：`<style="background-image:url(XXX.jpg)；background-position：XXX>"`

背景滚动：`<style="background-image:url(XXX.jpg)；background-attachment:scroll>"`

背景固定：`<style="background-image:url(XXX.jpg)；background-attachment:fixed>"`

设置背景颜色透明度（html5 新增）：<p style="background-color:rgba(255,0,0,0.5)">

背景简略写法：`<p style="background:一系列值">`，一系列值的顺序为：color、image、repeat、attachment、position

---

## 文本样式

### 段落样式

```
text-indent: 2em; /* em为当前字体宽度 */
padding:长度单位;
word-spacing : 长度单位 --- 针对英文,空格距离
letter-spacing : 长度单位 --- 字符之间的距离
text-transform : uppercase  --- 全部英文变为大写
text-tansform : lowercase  --- 全部英文变味小写
text-tansform : capitalize --- 首字母大写
```

```html
<p style="text-indent: 2em;text-transform : uppercase">
Hello, World
</p>
```

### 字体

字体系列，字体大类：font-family

```html
<p style="text-indent: 2em;font-family:fantasy;">
Hello, World
</p>
```

```
serif
sans-serif    没有多余的装饰线条
monospace    等宽的字体
cursive    手写的字体
fantasy    无法归类的，心型的
```

font-style：

```
italic    厂家做的好的倾斜
obique    浏览器计算产生的斜体。
```

`font-variant:small-caps;`: 小的大写字母。

字体加粗：

```
font-weight:normal;
font-weight:bold;
font-weight:900;
```

字体大小：

```
font-size:2em；// 1em是正常字体大小；10px 这种字体宽度单位是像素点
// 最好用EM，不是每个浏览器都支持绝对值的。
```

### 特殊效果

阴影效果：

```html
<p style="text-indent: 2em;text-shadow: 3px 5px 5px rgb(0,255,255)">
Hello, World
</p>
```

轮廓效果：

```html
<p style="text-indent: 2em;outline-color: red;outline-style: solid;">
Hello, World
</p>
```

---

## 列表和表格

列表样式：

```html
<ul style="list-style-type: square;">
  <li>apple</li>
  <li>banana</li>
</ul>
```

表格样式：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>表格边框样式</title>
<style>
/* 表格属性, 表示表格的两边框合并为一条 */
table {
    border-collapse: collapse;
}

/* 设置边框属性 */
table, td, th {
    border: 1px solid black;
}
</style>
</head>
<body>

<table>
  <tr>
    <th>Firstname</th>
    <th>Lastname</th>
  </tr>
  <tr>
    <td>Peter</td>
    <td>Griffin</td>
  </tr>
  <tr>
    <td>Lois</td>
    <td>Griffin</td>
  </tr>
</table>

<p><b>注意：</b> 如果没有指定 !DOCTYPE  border-collapse 属性在 IE8 及更早 IE 版本中是不起作用的。</p>

</body>
</html>
```

---

## CSS 定位

CSS 有两个最重要的基本属性，前端开发必须掌握：display 和 position。

display属性指定网页的布局。

position属性用来指定一个元素在网页上的位置（浏览器如何计算网页元素的位置）。

`static` 是position属性的默认值。如果省略position属性，浏览器就认为该元素是static定位。浏览器会按照源码的顺序，决定每个元素的位置。

`relative`、`absolute`、`fixed` 这三个属性值有一个共同点，都是相对于某个基点的定位，不同之处仅仅在于基点不同。

`sticky`跟前面四个属性值都不一样，它会产生动态效果，很像relative和fixed的结合：一些时候是relative定位（定位基点是自身默认位置），另一些时候自动变成fixed定位（定位基点是视口）。因此，它能够形成"动态固定"的效果。比如，网页的搜索工具栏，初始加载时在自己的默认位置（relative定位）。

---

## CSS 选择器

“选择器”指明了`{}`中的“样式”的作用对象，也就是“样式”作用于网页中的哪些元素

1. 通用元素选择器 `*`: 所有的标签都变色
2. 标签选择器：匹配所有使用`p`标签的样式 `p{color:red}`
3. id 选择器：匹配指定的标签  `#p2{color:red}`
4. class 类选择器：选择指定类，`.c1{color:red}`

---

## 参考

http://www.ruanyifeng.com/blog/2019/11/css-position.html

