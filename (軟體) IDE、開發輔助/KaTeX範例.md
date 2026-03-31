---
aliases:
date:
update:
author:
language:
sourceurl: https://blog.csdn.net/miao0967020148/article/details/78712811
tags:
---

# LaTeX 大括号公式和一般括号总结

## 大括号显示

$
\left\{
  \begin{array}{lr}
  x=\dfrac{3\pi}{2}(1+2t)\cos(\dfrac{3\pi}{2}(1+2t)), & \\
  y=s, & 0\leq s\leq L,|t|\leq1.\\
  z=\dfrac{3\pi}{2}(1+2t)\sin(\dfrac{3\pi}{2}(1+2t)), &
  \end{array}
\right.
$

## 对比括号一

$
\left\{
  \begin{array}{rcl}
  IF*{k}(\hat{t}*{k,m})=IF*{m}(\hat{t}*{k,m}), & \\
  IF*{k}(\hat{t}*{k,m}) \pm h= IF*{m}(\hat{t}*{k,m}) \pm h , & \\
  \left |IF'_{k}(\hat{t}_{k,m} - IF'_{m}(\hat{t}_{k,m} \right |\geq d , &
  \end{array}
\right.
$

## 常用的三种大括号写法

$
f(x)=\left\{
  \begin{aligned}
  x & = & \cos(t) \\
  y & = & \sin(t) \\
  z & = & \frac xy
  \end{aligned}
\right.
$

$
F^{HLLC}=\left\{
  \begin{array}{rcl}
  F_L       &      & {0      <      S_L}\\
  F^*_L     &      & {S_L \leq 0 < S_M}\\
  F^*_R     &      & {S_M \leq 0 < S_R}\\
  F_R       &      & {S_R \leq 0}
  \end{array} \right.
$

$
f(x)=
\begin{cases}
0& \text{x=0}\\
1& \text{x!=0}
\end{cases}
$

$
\begin{gathered}
\begin{matrix} 0 & 1 \\ 1 & 0 \end{matrix}
\quad
\begin{pmatrix} 0 & -i \\ i & 0 \end{pmatrix}
\quad
\begin{bmatrix} 0 & -1 \\ 1 & 0 \end{bmatrix}
\quad
\begin{Bmatrix} 1 & 0 \\ 0 & -1 \end{Bmatrix}
\quad
\begin{vmatrix} a & b \\ c & d \end{vmatrix}
\quad
\begin{Vmatrix} i & 0 \\ 0 & -i \end{Vmatrix}
\end{gathered}
$

## 功能 语法 显示

不好看 $\frac{1}{2}$

好一点 $\left( \frac{1}{2} \right)$

圆括号，小括号 $\left( \frac{a}{b} \right)$

方括号，中括号 $\left[ \frac{a}{b} \right]$

---

# LaTeX- 数学符号（二）矩阵与大括号

https://www.acwing.com/blog/content/3067/

## 矩阵

```markdown
$
\left+\+左括号
\begin{matrix}
第一行第一个数字 & 第一行第二个数字 & ... & 最后要加 \\\
第二行第一个数字 & 第二行第二个数字 & ... & 最后要加 \\\
\end{matrix}
\right+\+右括号
$
```

1. 若没有括号，第二行和倒数第二可不加
2. 左括号和右括号若是大括号，前面需要加两个\
3. 左括号和右括号可不同
4. 同行相邻内容之间一定要加&，表示该位置内容结束
5. 中间内容的结尾一定要加\\\，表示该行结束
6. 内容不一定是数字，可以加一些奇奇怪怪的字符，但貌似并没有卵用
7. 矩阵可以压行，但要保证格式一定对

无括号矩阵：

$
E=
\begin{matrix}
    1 & 0 & \cdots & 0 \\
    0 & 1 & \cdots & 0 \\
    \vdots & \vdots & \vdots & \vdots \\
    0 & 0 & \cdots & 1 \\
\end{matrix}
$

小括号矩阵：

$
E= \left(
\begin{matrix}
    1 & 0 & \cdots & 0 \\
    0 & 1 & \cdots & 0 \\
    \vdots & \vdots & \vdots & \vdots \\
    0 & 0 & \cdots & 1 \\
\end{matrix}
\right)
$

中括号矩阵：

$
E= \left[
\begin{matrix}
    1 & 0 & \cdots & 0 \\
    0 & 1 & \cdots & 0 \\
    \vdots & \vdots & \vdots & \vdots \\
    0 & 0 & \cdots & 1 \\
\end{matrix}
\right]
$

大括号矩阵：

$
E= \left\{
\begin{matrix}
    1 & 0 & \cdots & 0 \\
    0 & 1 & \cdots & 0 \\
    \vdots & \vdots & \vdots & \vdots \\
    0 & 0 & \cdots & 1 \\
\end{matrix}
\right\}
$

左小括号，右中括号矩阵：

$
E= \left(
\begin{matrix}
    1 & 0 & \cdots & 0 \\
    0 & 1 & \cdots & 0 \\
    \vdots & \vdots & \vdots & \vdots \\
    0 & 0 & \cdots & 1 \\
\end{matrix}
\right]
$

## 大括号

这个不是很好画，容易画的奇奇怪怪的

```markdown
\left+\+左括号
\begin{aligned}
内容 & 内容 & ... & 最后要加 \\\
内容 & 内容 & ... & 最后要加 \\\
\end{aligned}
\right+\+右括号
```

看起来和矩阵的画法没啥区别，对吧？
但实际上，如果你不加&，或者加错了&，这个就会自动向右对齐。。
就很难看。。
就像这个亚子 ↓

$
\left\{
\begin{aligned}
    x = \cos(t) \\
    y = \sin(t) \\
    z = \frac xy \\
\end{aligned}
\right.
$

哦~ 是因为没加& ~
然后你瞎加了几个&，就变成了这个亚子 ↓

$
\left\{
\begin{aligned}
    x & = & \cos(t) \\
    y & = & \sin(t) \\
    z & = & \frac xy \\
\end{aligned}
\right.
$

正确的 LaTeX 代码：

$
\left\{
\begin{aligned}
    x = & \cos(t) \\
    y = & \sin(t) \\
    z = & \frac xy \\
\end{aligned}
\right.
$

$ \vdots $
