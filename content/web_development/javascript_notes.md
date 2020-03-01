---
title: "JavaScript notes"
date: 2019-11-20 23:01
tag: web development
---

[TOC]

---

## 简介

- HTML          内容
- CSS           格式
- JavaScript    动作

Java 与 Javascript没有关系！
Javascript 是解释性语言
Javascript 源代码放在HTML里面

`script` 可以在 `head` 或 `body` 里：

- `head` 里：函数定义、变量定义
- `body` 里：实际动作

例子：

```html
<html>
<body onLoad = "alert('hi')">  注意用单引号
<script>
    document.write("Hello World!");
</script>
</body>
</html>
```

```html
<html>
<body>
<script>
     var hello = "Hello";
     document.write(hello);
</script>
</body>
</html>
```

JS 的变量都为 `var` 声明;（变量本身没有类型，但是值有类型）

运算符基本和Java一致；注释也基本一致；循环和判断语句也基本一样。

`for` 循环不带数据类型的声明。

注意： **JS语句执行完后才加载html** 。

---

## JavaScript 面向对象

JS中有对象的概念，但是它 **没有类的概念** ，不是先设计类再制造对象。

```js
var o = new Object();
var ciclr = {x:0, y:0, radius:2};
var book = new Object();
book.title = "HTML5秘籍";  // 可以动态添加属性
book.translator = "李松峰";
book.chapter1 = new Object();
book.chapter1.title = "HTML5简介";
// 删除对象属性
delete book.chapter1;
book.chapter1 = null;
// 遍历所有属性
for (var x in o) ...
```

```html
<html>
<head>
    <meta charset=utf-8>
</head>
<body>
<script>
var o = new Object();
o.name = "John";
o.age = 23;
o.salary = 300;
for (var x in o) {
    alert(x+"="o[x]);
}
</script>
</body>
</html>
```

```js
// 构造函数
function Rect(w, h) {
    this.width = w;
    this.height = h;
    this.area = function() {return this.width * this.height; };
}
var r = new Rect(5, 10);
alert(r.area());
```

### Prototype

`prototype` 是构造函数的一个属性, 该属性指向一个对象. 而这个对象将作为该构造函数所创建的所有实例的基引用(base reference),  **可以把对象的基引用想像成一个自动创建的隐藏属性** . 当访问对象的一个属性时, 首先查找对象本身, 找到则返回; 若不, 则查找基引用指向的对象的属性(如果还找不到实际上还会沿着 **原型链** 向上查找,  直至到根). 只要没有被覆盖的话, 对象原型的属性就能在所有的实例中找到.原型默认为 `Object` 的新实例, 由于仍是对象, 故可以给该对象添加新的属性。

```js
// 原型对象
// 对象的prototype属性指定了它的原型对象，可以用 . 运算符直接读它的对象属性；
function Person() {}  // function 也是对象

Person.prototype.name = "Nico";
Person.prototype.age = 29;
Person.prototype.job = "Coder";
Person.prototype.sayName = function() {
  alert(this.name);
}

// 原型中的属性是共享的
var person1 = new Person();
person1.sayName();
var person2 = new Person();
person2.sayName();
alert(person1.sayName == person2.sayName);

// 组合原型和构造方法
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ["Peter", "John"];
}
Person.prototype = {
  constructor: Person,
  sayName: function() {
    alert(this.name);
  }
};
```

---

## 浏览器中的JavaScript

- 浏览器的全局对象是 `window`
- 所有全局的变量实际上是 `window` 的成员

```html
<html>
<head>
    <meta charset=utf-8>
</head>
<body>
<script>
var value = 12;
alert(window.value);
</script>
</body>
</html>
```

- `window.document` 表示浏览器窗口中的HTML页面
- `document.write()` 将内容写入页面
- 页面中的元素就是 `document` 里的成员

```js
for (x in document)
    document.write(x+"<br/>");
```

## HTML 中的 Javascript

HTML 文件中的 JavaScript:

- 在 `<script></script>` 标记中
- 在 `<script>` 的 `src` 属性或 `archive` 指定的外部文件中
- 在某个HTML标记的事件处理器中

外部JS文件:

- `<script src="util.js"></script>`
- 一个纯粹的代码文件，没有HTML标记

事件处理器:

- `<p onMouseOver="alert('hi');">`

```html
<p onMouseOver="alert('hi');" onMouseOut="alert('bye');">
一个段落
</p>
```

`body` 事件:

- `onLoad`
- `onUnload`

简单对话框:

- `alert()`
- `confirm()`
- `prompt()`

```js
if (confirm("还要继续吗？")) {
    alert("继续");
} else {
    alert("再见");
}
var name = prompt("你的名字是:");
alert(name);
```

状态栏:

- `status =`
- `defaultStatus =`

```js
<p onMouseOver="status='Hello';" onMouseOut="status='';">
Hello
</p>
```

定时器：

- `setInterval`

window的控制方法:

- `window.open()`
   + `w=window.open("smallwin.html", "smallwin", "width=400,height=350,status=yes,resizable=yes"); // 打开新窗口`
   + `w.close();`
- `window.close(); // 把自己关掉`

location对象:

- `window.location` 代表当前文档的URL
   + `alert(location);`
   + `location="http://www.google.com";`

---

## DOM, 文档对象模型

DOM -- Document Object Model

`document` 对象的成员提供了HTML文档的信息


`document` 的成员

- `anchors[]`    链接
- `forms[]`        表单
- `images[]`
- `cookie`
- `title`
- `bgColor`
- `fgColor`
- `linkColor`
- `alinkColor`
- `vlinkColor`

## 图像

- `image` 对象的 `src` 可以改写以装入一幅新图片
- 可以创建 `Image()` 对象来提前装载图片
- `onLoad` 事件表明图片装载完成

```html
<body onLoad="setInterval('show()', 200)">
<img name="anm" src="0.jpg" width=300></img>
<script>
var images = new Array(6);
var index = 1;
for (var i = 0; i < 6; ++i) {
    images[i] = new Image();
     images[i].src = i + ".jpg";
}
function show() {
    document.anm.src = images.src = images[index].src;
     index = (index + 1) % 6;
}
</script>
```

## 事件

- `onLoad` / `onUnload`
- `onMouseOver` / `onMouseOut`
- `onClick` / `onDblClick`
- `onSubmit`
