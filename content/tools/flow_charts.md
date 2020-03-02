---
title: "程序流程图"
date: 2019-07-09 00:01
---

[TOC]

### 什么是程序流程图

- 以简单的图形符号来表达问题解决步骤的示意图，亦称为框图。
- 流程图是问题求解的最基本、最重要的程序分析技术。
- 程序设计时先画出流程图，往往可以事半功倍。

---

### 常用流程图图形符号

使用 [Mermaid](https://mermaidjs.github.io/) 进行绘制。

下面各种框的图形只是个人喜好，实际使用根据需要调整。

#### 起止框

标准流程的开始、结束。

```
<div class="mermaid">
graph LR
  a(START)
</div>
```

<div class="mermaid">
graph LR
  a(START)
</div>

#### 处理框

算法、程序要执行的处理操作。

```
<div class="mermaid">
graph LR
  a[PROCESS]
</div>
```

<div class="mermaid">
graph LR
  a[PROCESS]
</div>

#### 判断框

判断条件是否成立。

```
<div class="mermaid">
graph LR
  a{if a > b}
</div>
```

<div class="mermaid">
graph LR
  a{if a > b}
</div>

#### 输入输出框

表示数据的输入、输出。（一般用 平行四边形 ，Mermaid 里没找到）

```
<div class="mermaid">
graph LR
  a>INPUT]
</div>
```

<div class="mermaid">
graph LR
  a>INPUT]
</div>

---

### 流程图设计要求

1. 单入口、单出口
2. 无死语句（即永远不能被执行到的语句）
3. 无死循环（即程序永远不能停止）

### 常见流程图结构

- 顺序结构
- 选择结构（如 if 语句）
- 循环结构（如 while 语句）

#### if 语句

```
<div class="mermaid">
graph TB
  id1{if a > b}--True-->id2[Process A]
  id1--False-->id3[Process B]
  id2-->id4[Process C]
  id3-->id4[Process C]
</div>
```

<div class="mermaid">
graph TB
  id1{if a > b}--True-->id2[Process A]
  id1--False-->id3[Process B]
  id2-->id4[Process C]
  id3-->id4[Process C]
</div>

#### while 循环语句

```
<div class="mermaid">
graph LR
  id1(START)-->id2>INPUT DATA]
  id2-->id3[INIT]
  id3-->id4{while ...}
  id4--True-->id5[UPDATE]
  id5-->id4
  id4--False-->id6[POSTPROCESS]
  id6-->id7>OUTPUT DATA]
  id7-->id8(END)
</div>
```

<div class="mermaid">
graph LR
  id1(START)-->id2>INPUT DATA]
  id2-->id3[INIT]
  id3-->id4{while ...}
  id4--True-->id5[UPDATE]
  id5-->id4
  id4--False-->id6[POSTPROCESS]
  id6-->id7>OUTPUT DATA]
  id7-->id8(END)
</div>
