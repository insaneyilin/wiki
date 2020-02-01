---
title: "Jupyter Notebook notes"
date: 2019-10-31 00:01
---

[TOC]

## 基本用法

Jupyter Notebook（又称IPython Notebook）是一个交互式的笔记本，使用浏览器作为界面，向后台的IPython服务器发送请求，并显示结果。在浏览器的界面中使用单元(Cell)保存各种信息。Cell有多种类型，经常使用的有表示格式化文本的Markdown单元，和表示代码的Code单元。

启动 jupyter server 的命令：

```
jupyter notebook
```

默认的网页地址为 `http://localhost:8888/`。

在远程服务器上启动 jupyter notebook：

```
jupyter notebook --no-browser --port=8889
```

笔记本文件格式为 `.ipynb`，实质是 `json` 文件。

常用的 notebook cell：

- markdown cell
- code cell

常用快捷键：

- `ctrl+enter` ：运行当前 code cell
- `alt+enter`：运行当前 code cell，并在下面再插入一个 code cell
- `h`：帮助提示
- `f`：搜索


