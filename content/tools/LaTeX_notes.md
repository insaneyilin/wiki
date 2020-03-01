---
title: "LaTeX memo"
date: 2019-09-27 00:01
---

[TOC]

---

## Github readme 或者网页中插入 LaTeX 公式图片

借助下面这个网站：

https://www.codecogs.com/latex/eqneditor.php

写完公式直接复制 html 代码插入到 markdown 文件中即可：

<img src="https://latex.codecogs.com/gif.latex?e^{i\pi}&space;&plus;&space;1&space;=&space;0" title="e^{i\pi} + 1 = 0" />

---

## 常用符号用法备忘

符号速查：[Wikipedia_LaTeX_symbols](http://ia.wikipedia.org/wiki/Wikipedia:LaTeX_symbols)

[PDF 文件](/wiki/attach/files/Wikipedia_LaTeX_symbols.pdf)

连加：

```
\sum
```

连乘：

```
\prod
```

花体字母：

```
\cal{}，使用\cal有时会导致后面的字母全部变成花体，记得用{}将前面相应的项括起来
```

数学公式中的花体字母：

```
\mathcal{}
```

下标正下方：

```
\sum \limits_{0 < i < 100}
```

点乘：

```
\cdot
```

运算符号：

```
乘号：\times, 除号：\div, 正负号：\pm
```

角符号：

```
\angle
```

无穷：

```
\infty
```

任意：

```
\forall,
```

存在：

```
\exists
```


范数：

```
\|
```

空格：

```
单独一个\是空格，\quad是一个m的宽度，\qquad是两个m的宽度
```

大于等于号：

```
\geq, \geqslant
```

小于等于号：

```
\leq, \leqslant
```

约等于号：

```
\approx
```

相似于：

```
\sim
```

恒等号：

```
\equiv
```

字母上方加横线（如均值）：

```
\bar{}
```

上划线、下划线：

```
\overline{}, \underline{}
```

字母上方加波浪线：

```
\tilde{}
```

字母上方加箭头（表示向量）：

```
\vec{}
```

字母上方加点：

```
\dot{}
```

黑体加斜体的公式

```
用\boldsymbol{}
```

行列式与矩阵：

```
\[
\left| \begin{array}{cccc}
1 & 6 & 9 \\
7 & 90 & f(x) \\
9 & \psi(x) & g(x)
\end{array} \right|
\]

其中 \left| 和  \right|  表示左右定界符。如果我们将|换成 (  )或 [  ]，就得到了矩阵。
在这里，行列式和矩阵都是中间对齐的，如果你想左对齐或右对齐，你将{cccc}换成{llll}(左对齐)或{rrrr}(右对齐)就行了。& 是对齐符号。l=left  c=center  r=right
```

<img src="https://latex.codecogs.com/gif.latex?\left|&space;\begin{array}{cccc}&space;1&space;&&space;6&space;&&space;9&space;\\&space;7&space;&&space;90&space;&&space;f(x)&space;\\&space;9&space;&&space;\psi(x)&space;&&space;g(x)&space;\end{array}&space;\right|" title="\left| \begin{array}{cccc} 1 & 6 & 9 \\ 7 & 90 & f(x) \\ 9 & \psi(x) & g(x) \end{array} \right|" />

方程组与分段函数：

方程组

```
\[
\begin{cases} 
\ u_{tt}(x,t)= b(t)\triangle u(x,t-4) & \\ 
\ \hspace{42pt}- q(x,t)f[u(x,t-3)]+te^{-t}\sin^2 x,  &  t \neq t_k; \\ 
\ u(x,t_k^+) - u(x,t_k^-) = c_k u(x,t_k), & k=1,2,3\ldots ; \\ 
\ u_{t}(x,t_k^+) - u_{t}(x,t_k^-) =c_k u_{t}(x,t_k), &  k=1,2,3\ldots \ . 
\end{cases} 
\]

& 是对齐符号，\\是换行符号，\hspace{距离} 插入任意空格，\neq不等于号，注意命令与后面的拉开距离，\ldots是省略符号。
```

<img src="https://latex.codecogs.com/gif.latex?\begin{cases}&space;\&space;u_{tt}(x,t)=&space;b(t)\triangle&space;u(x,t-4)&space;&&space;\\&space;\&space;\hspace{42pt}-&space;q(x,t)f[u(x,t-3)]&plus;te^{-t}\sin^2&space;x,&space;&&space;t&space;\neq&space;t_k;&space;\\&space;\&space;u(x,t_k^&plus;)&space;-&space;u(x,t_k^-)&space;=&space;c_k&space;u(x,t_k),&space;&&space;k=1,2,3\ldots&space;;&space;\\&space;\&space;u_{t}(x,t_k^&plus;)&space;-&space;u_{t}(x,t_k^-)&space;=c_k&space;u_{t}(x,t_k),&space;&&space;k=1,2,3\ldots&space;\&space;.&space;\end{cases}" title="\begin{cases} \ u_{tt}(x,t)= b(t)\triangle u(x,t-4) & \\ \ \hspace{42pt}- q(x,t)f[u(x,t-3)]+te^{-t}\sin^2 x, & t \neq t_k; \\ \ u(x,t_k^+) - u(x,t_k^-) = c_k u(x,t_k), & k=1,2,3\ldots ; \\ \ u_{t}(x,t_k^+) - u_{t}(x,t_k^-) =c_k u_{t}(x,t_k), & k=1,2,3\ldots \ . \end{cases}" />

分段函数

```
\[
q(x, t) =
\begin{cases}
(t-k+1)x^2, \quad \ \ & t \in \big( k-1, k - \dfrac{1}{2} \big], \\
(k-t)x^2, \quad \ \ & t \in \big( k - \dfrac{1}{2}, k \big],
\end{cases}
\]

\quad和\qquad用于插入固定长度的水平间距，\quad为插入当前字体尺寸大小的间距，\qquad是其两倍
```

<img src="https://latex.codecogs.com/gif.latex?q(x,&space;t)&space;=&space;\begin{cases}&space;(t-k&plus;1)x^2,&space;\quad&space;\&space;\&space;&&space;t&space;\in&space;\big(&space;k-1,&space;k&space;-&space;\dfrac{1}{2}&space;\big],&space;\\&space;(k-t)x^2,&space;\quad&space;\&space;\&space;&&space;t&space;\in&space;\big(&space;k&space;-&space;\dfrac{1}{2},&space;k&space;\big],&space;\end{cases}" title="q(x, t) = \begin{cases} (t-k+1)x^2, \quad \ \ & t \in \big( k-1, k - \dfrac{1}{2} \big], \\ (k-t)x^2, \quad \ \ & t \in \big( k - \dfrac{1}{2}, k \big], \end{cases}" />

公式换行与对齐

```
\begin{equation}
\begin{aligned}
X^{*}, L^{*} &= argmax_{X,L} \ p(X, L | Z, B) \\
&= argmax_{X, L} \ p(x_1) \prod_{i, k}p(x_i|x_k, b_{ik}) \prod_{i, j}p(z_{ij}|x_i, l_{j})
\end{aligned}
\end{equation}
```

<img src="https://latex.codecogs.com/gif.latex?\begin{aligned}&space;X^{*},&space;L^{*}&space;&=&space;argmax_{X,L}&space;\&space;p(X,&space;L&space;|&space;Z,&space;B)&space;\\&space;&=&space;argmax_{X,&space;L}&space;\&space;p(x_1)&space;\prod_{i,&space;k}p(x_i|x_k,&space;b_{ik})&space;\prod_{i,&space;j}p(z_{ij}|x_i,&space;l_{j})&space;\end{aligned}" title="\begin{aligned} X^{*}, L^{*} &= argmax_{X,L} \ p(X, L | Z, B) \\ &= argmax_{X, L} \ p(x_1) \prod_{i, k}p(x_i|x_k, b_{ik}) \prod_{i, j}p(z_{ij}|x_i, l_{j}) \end{aligned}" />

一种简单的参考文献写法(不使用 bibtex )：

```
% 引用参考文献
\cite{oxford1}
% 放在源文件末尾，\end{document} 之前
\begin{thebibliography}{99}
\bibitem{oxford1}Real-Time 3D Tracking and Reconstruction on Mobile Phones
\end{thebibliography}
```

使用 listings 环境插入代码并高亮显示：

```
\usepackage{mdwlist}
\begin{lstlisting}[language={[ANSI]C},keywordstyle=\color{blue!70},
commentstyle=\color{red!50!green!50!blue!50},frame=shadowbox,
rulesepcolor=\color{red!20!green!20!blue!20}]
// Finds the camera intrinsic and extrinsic parameters from
// several views of a calibration pattern
double calibrateCamera(
     InputArrayOfArrays objectPoints,
     InputArrayOfArrays imagePoints,
     Size imageSize,
     InputOutputArray cameraMatrix,
     InputOutputArray distCoeffs,
     ... );
// Finds the positions of internal corners of the chessboard
bool findChessboardCorners(
     InputArray image,
     Size patternSize,
     OutputArray corners,
     ... );
// Transforms an image to compensate for lens distortion
void undistort(
     InputArray src,
     OutputArray dst,
     InputArray cameraMatrix,
     InputArray distCoeffs,
     InputArray newCameraMatrix=noArray() );
\end{lstlisting}
```

