---
title: "Python Web 开发"
date: 2019-11-20 23:01
tag: python
---

[TOC]

---

## HTTP协议简介

在Web应用中，服务器把网页传给浏览器，实际上就是把网页的HTML代码发送给浏览器，让浏览器显示出来。而浏览器和服务器之间的传输协议是HTTP，所以：

- HTML是一种用来定义网页的文本，会HTML，就可以编写网页；
- HTTP是在网络上传输HTML的协议，用于浏览器和服务器的通信。

### HTTP请求流程

- 步骤1：浏览器首先向服务器发送HTTP请求，请求包括：
    + 方法：GET还是POST，GET仅请求资源，POST会附带用户数据；
    + 路径：/full/url/path；
    + 域名：由Host头指定：Host: www.sina.com.cn
    + 以及其他相关的Header；
    + 如果是POST，那么请求还包括一个Body，包含用户数据。
- 步骤2：服务器向浏览器返回HTTP响应，响应包括：
    + 响应代码：200表示成功，3xx表示重定向，4xx表示客户端发送的请求有错误，5xx表示服务器端处理时发生了错误；
    + 响应类型：由 `Content-Type` 指定，例如：`Content-Type: text/html;charset=utf-8` 表示响应类型是HTML文本，并且编码是 `UTF-8` ， `Content-Type: image/jpeg` 表示响应类型是JPEG格式的图片；
    + 以及其他相关的Header；
    + 通常服务器的HTTP响应会携带内容，也就是有一个Body，包含响应的内容，网页的HTML源码就在Body中。
- 步骤3：如果浏览器还需要继续向服务器请求其他资源，比如图片，就再次发出HTTP请求，重复步骤1、2。

Web采用的HTTP协议采用了非常简单的请求-响应模式，从而大大简化了开发。当我们编写一个页面时，我们只需要在HTTP响应中把HTML发送出去，不需要考虑如何附带图片、视频等，浏览器如果需要请求图片和视频，它会发送另一个HTTP请求，因此， **一个HTTP请求只处理一个资源** 。

### HTTP格式

每个HTTP请求和响应都遵循相同的格式，一个HTTP包含Header和Body两部分，其中Body是可选的。

HTTP协议是一种文本协议，所以，它的格式也非常简单。HTTP GET请求的格式：

```
GET /path HTTP/1.1
Header1: Value1
Header2: Value2
Header3: Value3
```

每个Header一行一个，换行符是 `\r\n` 。

HTTP POST请求的格式：

```
POST /path HTTP/1.1
Header1: Value1
Header2: Value2
Header3: Value3

body data goes here...
```

当遇到连续两个\r\n时，Header部分结束，后面的数据全部是Body。

HTTP响应的格式：

```
200 OK
Header1: Value1
Header2: Value2
Header3: Value3

body data goes here...
```

HTTP响应如果包含body，也是通过\r\n\r\n来分隔的。请再次注意，Body的数据类型由Content-Type头来确定，如果是网页，Body就是文本，如果是图片，Body就是图片的二进制数据。

当存在Content-Encoding时，Body数据是被压缩的，最常见的压缩方式是gzip，所以，看到 `Content-Encoding: gzip` 时，需要将Body数据先解压缩，才能得到真正的数据。压缩的目的在于减少Body的大小，加快网络传输。

---

## HTML CSS JavaScript 简介

HTML 不是编程语言，是标记语言。

HTML定义了一套语法规则，来告诉浏览器如何把一个丰富多彩的页面显示出来。

CSS是Cascading Style Sheets（层叠样式表）的简称，CSS用来控制HTML里的所有元素如何展现。

JavaScript虽然名称有个Java，但它和Java真的一点关系没有。JavaScript是 **为了让HTML具有交互性而作为脚本语言添加的** ，JavaScript既可以内嵌到HTML中，也可以从外部链接到HTML中。

要学习Web开发，首先要对HTML、CSS和JavaScript作一定的了解。HTML定义了页面的内容，CSS来控制页面元素的样式，而JavaScript负责页面的交互逻辑。

---

## WSGI：Web Server Gateway Interface

一个Web应用的工作流程：

- 浏览器发送一个HTTP请求；
- 服务器收到请求，生成一个HTML文档；
- 服务器把HTML文档作为HTTP响应的Body发送给浏览器；
- 浏览器收到HTTP响应，从HTTP Body取出HTML文档并显示。

最简单的Web应用就是先把HTML用文件保存好，用一个现成的HTTP服务器软件，接收用户请求，从文件中读取HTML，返回。Apache、Nginx、Lighttpd等这些常见的静态服务器就是干这件事情的。

如果要动态生成HTML，就需要把上述步骤自己来实现。不过，接受HTTP请求、解析HTTP请求、发送HTTP响应都是苦力活，如果我们自己来写这些底层代码，还没开始写动态HTML呢，就得花个把月去读HTTP规范。

正确的做法是 **底层代码由专门的服务器软件实现** ，我们用Python专注于生成HTML文档。因为我们不希望接触到TCP连接、HTTP原始请求和响应格式，所以，需要一个统一的接口，让我们专心用Python编写Web业务。

这个接口就是WSGI：Web Server Gateway Interface。

WSGI接口定义非常简单，它只要求Web开发者实现一个函数，就可以响应HTTP请求。我们来看一个最简单的Web版本的“Hello, web!”：

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return [b'<h1>Hello, web!</h1>']
```

- `environ`：一个包含所有HTTP请求信息的dict对象；
- `start_response`：一个发送HTTP响应的函数。

Python内置了一个WSGI服务器，这个模块叫wsgiref，它是用纯Python编写的WSGI服务器的参考实现。所谓“参考实现”是指该实现完全符合WSGI标准，但是不考虑任何运行效率，仅供开发和测试使用。

无论多么复杂的Web应用程序，入口都是一个WSGI处理函数。HTTP请求的所有输入信息都可以通过 `environ` 获得，HTTP响应的输出都可以通过 `start_response()` 加上函数返回值作为Body。

复杂的Web应用程序，光靠一个WSGI函数来处理还是太底层了，我们需要在WSGI之上再抽象出 **Web框架** ，进一步简化Web开发。

---

## Web框架

Python 有很多 Web 框架，如 Django, Tornado, Flask 等。Flask 很轻量，适合快速入门。下面简单介绍 Flask。

我们用 Flask 写一个 `app.py`，处理3个URL，分别是：

- GET /：首页，返回Home；
- GET /signin：登录页，显示登录表单；
- POST /signin：处理登录表单，显示登录结果。

注意，同一个URL `/signin` 分别有GET和POST两种请求，映射到两个处理函数中。

Flask通过Python的装饰器在内部自动地把URL和函数给关联起来。

```python
from flask import Flask
from flask import request

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def home():
    return '<h1>Home</h1>'

@app.route('/signin', methods=['GET'])
def signin_form():
    return '''<form action="/signin" method="post">
              <p><input name="username"></p>
              <p><input name="password" type="password"></p>
              <p><button type="submit">Sign In</button></p>
              </form>'''

@app.route('/signin', methods=['POST'])
def signin():
    # 需要从request对象读取表单内容：
    if request.form['username']=='admin' and request.form['password']=='password':
        return '<h3>Hello, admin!</h3>'
    return '<h3>Bad username or password.</h3>'

if __name__ == '__main__':
    app.run()
```

常见的Python Web框架：

- Django：全能型Web框架；
- web.py：一个小巧的Web框架；
- Bottle：和Flask类似的Web框架；
- Tornado：Facebook的开源异步Web框架。

--

## 使用模板

Web App不仅仅是处理逻辑，展示给用户的页面也非常重要。在函数中返回一个包含HTML的字符串，简单的页面还可以，但是，想想新浪首页的6000多行的HTML，你确信能在Python的字符串中正确地写出来么？

使用模板，我们 **需要预先准备一个HTML文档，这个HTML文档不是普通的HTML，而是嵌入了一些变量和指令，然后，根据我们传入的数据，替换后，得到最终的HTML，发送给用户** 。

这是一种 MVC 模式：MVC：Model-View-Controller，中文名“模型-视图-控制器”。

- Python处理URL的函数就是C：Controller，Controller负责业务逻辑，比如检查用户名是否存在，取出用户信息等等；
- 包含变量 `{{ name }}` 的模板就是V：View，View负责显示逻辑，通过简单地替换一些变量，View最终输出的就是用户看到的HTML。
- Model是用来传给View的，这样View在替换变量的时候，就可以从Model中取出相应的数据。Model 用来表示数据。

```python
from flask import Flask, request, render_template

app = Flask(__name__)

# 使用 render_template 渲染模板
@app.route('/', methods=['GET', 'POST'])
def home():
    return render_template('home.html')

@app.route('/signin', methods=['GET'])
def signin_form():
    return render_template('form.html')

@app.route('/signin', methods=['POST'])
def signin():
    username = request.form['username']
    password = request.form['password']
    if username=='admin' and password=='password':
        return render_template('signin-ok.html', username=username)
    # 传入参数 message, username 到对应模板中
    return render_template('form.html', message='Bad username or password', username=username)

if __name__ == '__main__':
    app.run()
```

Flask默认支持的模板是 `jinja2`。

`home.html` :

```html
<html>
<head>
  <title>Home</title>
</head>
<body>
  <h1 style="font-style:italic">Home</h1>
</body>
</html>
```

`form.html`：

```html
<html>
<head>
  <title>Please Sign In</title>
</head>
<body>
  {% if message %}
  <p style="color:red">{{ message }}</p>
  {% endif %}
  <form action="/signin" method="post">
    <legend>Please sign in:</legend>
    <p><input name="username" placeholder="Username" value="{{ username }}"></p>
    <p><input name="password" placeholder="Password" type="password"></p>
    <p><button type="submit">Sign In</button></p>
  </form>
</body>
</html>
```

`signin-ok.html` :

```html
<html>
<head>
  <title>Welcome, {{ username }}</title>
</head>
<body>
  <p>Welcome, {{ username }}!</p>
</body>
</html>
```

Flask 文件组织结构：

```
.
|____app.py
|____templates/
        |____form.html
        |____home.html
        |____signin-ok.html
```

在Jinja2模板中，我们用 `{{ name }}` 表示一个需要替换的变量。很多时候，还需要循环、条件判断等指令语句，在Jinja2中，用 `{% ... %}` 表示指令。

比如循环输出页码：

```
{% for i in page_list %}
    <a href="/page/{{ i }}">{{ i }}</a>
{% endfor %}
```

常用其他模板：

- Mako：用<% ... %>和${xxx}的一个模板；
- Cheetah：也是用<% ... %>和${xxx}的一个模板；
- Django：Django是一站式框架，内置一个用{% ... %}和{{ xxx }}的模板。


