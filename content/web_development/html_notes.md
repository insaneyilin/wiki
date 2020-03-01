---
title: "HTML notes"
date: 2019-11-20 23:01
tag: web development
---

[TOC]

---

## HTML: Hyper Text Markup Language

不算是编程语言，属于标记语言。

SGML，一种很强大但是很复杂的标记语言，比html要早。

SGML 最为强大和古老，XML 是 SGML 的一个子集，HTML 最初也试图成为 SGML 的一个子集，但 HTML 4 以前并不严格符合 SGML 的标准。后来出现了 HTML4，XHTML （符合 XML 标准的 HTML），这两者都符合 SGML 的要求。

到现如今，W3C 在制定 HTML5 标准时，希望摆脱 SGML 的一些无用的功能和声明，并延续 HTML4 的风格，而非严格的 XHTML 的风格。

---

一个html的例子：

```html
<html>
<body>
<!--这是一段注释。注释不会在浏览器中显示。-->
<p>这是一段普通的段落。</p>
<p>注释的写法如下：</p>
<p> &lt!-- This is a comment. --&gt </p>
</body>
</html>
```

## HTML 与 CSS

通过使用 HTML4.0，所有的格式化代码均可移出 HTML 文档，然后移入一个独立的样式表。

HTML是用来表达内容、结构“是什么”；

样式由样式表CSS负责。

---

## HTML 基本用法

### 段落与标题

```
<p> </p>                 段落，段落间空一行；
<br/>                    断行，换行，没有结束符，可直接写成<br>；
<h1> </h1>               第一级标题，前后自动空一行，无需加<br>；
<h2> </h2>               第二级标题，共支持6级标题；
<wbr> </wbr>             英文单词可从中间断开，对中文没意义；
<hgroup> </hgroup>       告诉浏览器连续的二级标题，中间为h1，h2等
```

### 文字样式

```
<b>关键词</b>，加粗(bold)
<i>斜体字</i>
<tt>等宽的西方字体<tt>
<small>字体变小</small>
<del>字一</del><ins>字二</ins>，标记删除字一，用字二
<s>告诉浏览器这些内容是不被提倡的</s>
<sup>上标</sup>   <sub>下标</sub>
<mark>标亮字</mark>
// 文字样式一般都可以在css中实现

<em>emphasize强调</em>；
<strong>着重</strong>；
<dfn>definition定义</dfn>；
<code>代码，告诉浏览器该部分为源代码(较短的)，对中文意义不大，对英文有用</code>；
<samp>sample例子代码</samp>；
<kbd>keyboard用户输入的</kbd>；
<var>variable变量</var>；
<cite>cite该部分为从别处引用</cite>；

<address></address>               地址标签
<blockquote></blockquote>         缩进，可以嵌套
<q></q>                           小引用
<code></code>                     代码
<pre></pre>                       预格式化
```

再次记住，html不是用于表述格式(样式)的，是用来描述结构的(“是什么”)。

插入代码的例子：

```
<code>
<pre>
int main() {
     printf("hello\n")
     return 0;
}
</pre>
</code>
```

<code>
<pre>
int main() {
     printf("hello\n")
     return 0;
}
</pre>
</code>

```
<hr>水平线（horizontal ruler）      // 不用加</hr>
<hr width=50%>         宽度50%
<hr width=50>          宽度50
<hr width=50% align=left size=10>     宽度50%
```

HTML5不需要属性的值加双引号了;

现在的属性提倡用CSS来做。

```
<bdo dir=rtl> 文字从右向左排</bdo>；
<bdo dir=rtl><bdi>将颠倒的文字颠倒过来</bdi></bdo>；

&lt; (注意有分号作为结束)小于号
&gt; 大于号
&amp; &符号
&nbsp; 空格(non-nbreakable space)
// 还有很多特殊字符，用的时候自己去查。
```

### 列表

```
<ul></ul>  不排序的列表, unordered list
<ol></ol>  排序的列表，会加数字
<dl></dl>  类似于字典，其中可以用<dt></dt>和<dd></dd>添加词条、解释词条
<li></li> list item

<ul>
     <li>红茶</li>
     <li>绿茶</li>
     <li>可乐</li>
</ul>
<ol start = 2>
     <li>红茶</li>
     <li>绿茶</li>
     <li>可乐</li>
     <li>咖啡</li>
          <ul>
               <li>加糖</li>
               <li>不加糖</li>
          </ul>
</ol>
<dl>
     <dt>方糖</dt>
     <dd>方的糖，甜的</dd>
</dl>
```

### 图片

```
// 用img添加图片，对于html来说，图片是一个“字符”（排版角度）
<img src="文件名/网站地址名图片" width=800 heigt=600 alt="图片未正常显示时所显示的alternative文字" />
```

### 链接

```
<iframe src="http://news.163.com" width = 400></iframe>
// 用于把另外的页面插入进来

<a href="http://www.baidu.com">链接名</a>
<p id="here">  // 为段落加标记
<a href="#here">here</a>  // 点击后到达上面那个标记为here的段落
<a href="you.html#here">here</a>  // 点击后到达you.html中标记为here的段落

// 在新窗口中打开超链接
<a href ="http://news.163.com" target=_blank></a>

// usemap可以为图片中的某个区域添加超链接
<img src=""  usemap="#map"/>
<map name="map">.......
```

### 表格

```
<table border="1">
     <caption>表格标题</caption>
     <tr>
          <th>OS</th>
          <th>Chinese</th>
          <th>French</th>
     </tr>
     <tr>
          <td>hehe</td>
          <td>yes</td>
          <td>no</td>
     </tr>
</table>

th: 表头

tr: table row

td: 单元格

colspan="3": 延展三格
<tr colspan="3"></tr>
```

