---
title: "Matrix Calculus Notes"
date: 2019-10-04 18:01
---

[TOC]

矩阵求导的一些笔记。

---

## 单变量求导规则(Scalar derivative rules)

### 常见基本导数形式与规则

![Basic rules of derivatives](/wiki/attach/images/matrix_calculus_notes/scalar_derivative_rules.png)

### 链式法则(the chain rule)

设 $y = f(u)$，$u = g(x)$，那么 $y = f(g(x))$ 。

则 $y$ 关于 $x$ 的导数可以用链式法则来求出：

$$\frac{\partial{y}}{\partial{x}} = \frac{\partial{y}}{\partial{u}} \frac{\partial{u}}{\partial{x}}$$

---

## 多变量函数求导

对于 $f(x, y) = 3 x^2 y$，分别对自变量 x, y 求导，则得到了 $f(x, y)$ 的梯度（gradient）：

$$ \nabla{f}(x, y) = [\frac{\partial f}{\partial x}, \frac{\partial f}{\partial x}] = [6xy, 3x^2] $$

---

## 矩阵求导

设自变量为

$$ \mathbf{x} = \left[ \begin{array} \
x_1 \\\
x_2 \\\
... \\\
x_n
\end{array} \right] $$

因变量为

$$ \mathbf{y} = \left[ \begin{array} \
y_1 \\\
y_2 \\\
... \\\
y_m
\end{array} \right] $$

且

$$ y_1 = f_1(\mathbf{x}) \\\
y_2 = f_2(\mathbf{x}) \\\
... \\\
y_m = f_m(\mathbf{x})
$$

定义 y 关于 x 的导数为 y 关于 x 的 Jacobian 矩阵：

![Jacobian matrix](/wiki/attach/images/matrix_calculus_notes/multi_var_jacobian.png)

## 矩阵求导链式法则

![Matrix chain rule](/wiki/attach/images/matrix_calculus_notes/matrix_chain_rule.png)

注意矩阵乘法不具有交换性。

---

## 相关资源

[Old and New Matrix Algebra Useful for Statistics](/wiki/attach/images/matrix_calculus_notes/minka-matrix.pdf), Thomas P. Minka

The Matrix Cookbook, Kaare Brandt Petersen and Michael Syskind Pedersen

https://en.wikipedia.org/wiki/Matrix_calculus

