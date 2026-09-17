+++
title = '高数基础复习笔记（6）：反函数综合例题'
date = '2026-09-17T12:00:00+08:00'
lastmod = '2026-09-17T12:00:00+08:00'
draft = false
description = '用一道反函数题串起指数、换元、求根公式、根号化简、舍根与自然对数。'
summary = '用一道反函数题串起指数、换元、求根公式、根号化简、舍根与自然对数。'
tags = ['高等数学', '学习笔记']
isCJKLanguage = true
ShowToc = true
TocOpen = true
UseHugoToc = true
studyseries = 'calculus-basics'
studyorder = 6
+++

用一道反函数题串起指数、换元、求根公式、根号化简、舍根与自然对数。

> 本文是「高数基础复习笔记」系列第 6 篇，对应原笔记第 21 节。除非特别说明，只讨论**实数范围**。
>
> [总目录与公式速查]({{< relref "/posts/calculus-basics.md" >}})。保留原章节编号，便于对照复习；本篇会随学习进度持续补充。

## 21. 综合例题：求 y=(e^x-e^{-x})/2 的反函数 {#section-21}

原函数为：

$$
\boxed{f(x)=\frac{e^x-e^{-x}}2},\qquad x\in\mathbb R.
$$

它也叫双曲正弦函数，记作 $\sinh x$，与 $\sin x$ 不同。这道题不需要提前掌握双曲函数知识。

### 21.1 先判断能不能有反函数

$x$ 增大时，$e^x$ 严格增大，$e^{-x}$ 严格减小，所以 $-e^{-x}$ 严格增大。因此 $f(x)$ 严格增大，不同输入对应不同输出。

下面的推导还会证明：每一个实数 $y$ 都能找到唯一对应的 $x$。因此原函数值域为 $\mathbb R$，并存在反函数。

### 21.2 第一步：消掉分母 2

等式两边同乘 $2$：

$$
y=\frac{e^x-e^{-x}}2
\quad\Longrightarrow\quad
2y=e^x-e^{-x}.
$$

### 21.3 第二步：负指数变成倒数

因为 $e^{-x}=1/e^x$，所以：

$$
2y=e^x-\frac1{e^x}.
$$

### 21.4 第三步：两边同乘 e^x

$e^x>0$，不会为零，因此这一步可逆：

$$
2ye^x=e^x\cdot e^x-\frac1{e^x}\cdot e^x=e^{2x}-1.
$$

用到了 $e^x\cdot e^x=e^{2x}$ 和 $(1/e^x)\cdot e^x=1$。

### 21.5 第四步：移项，整理成一边为零

$$
0=e^{2x}-1-2ye^x.
$$

交换等式左右、整理项的顺序：

$$
\boxed{e^{2x}-2ye^x-1=0}.
$$

### 21.6 第五步：换元，把复杂式子整体当成未知数

设：

$$
t=e^x,\qquad t>0.
$$

由于 $e^{2x}=(e^x)^2=t^2$，方程变成：

$$
\boxed{t^2-2yt-1=0}.
$$

这里求的是 $t$，把 $y$ 暂时看作一个给定的数。因此：

$$
a=1,\qquad b=-2y,\qquad c=-1.
$$

### 21.7 第六步：代入求根公式

$$
\begin{aligned}
t
&=\frac{-b\pm\sqrt{b^2-4ac}}{2a}\\
&=\frac{-(-2y)\pm\sqrt{(-2y)^2-4\times1\times(-1)}}{2\times1}\\
&=\frac{2y\pm\sqrt{4y^2+4}}2.
\end{aligned}
$$

逐项看：$-(-2y)=2y$，$(-2y)^2=4y^2$，$-4\times1\times(-1)=+4$。

判别式 $4y^2+4>0$，所以对任意实数 $y$，这个二次方程都有两个不同实根。

### 21.8 第七步：根号内提 4，根号外得到 2

$$
\sqrt{4y^2+4}
=\sqrt{4(y^2+1)}
=\sqrt4\sqrt{y^2+1}
=2\sqrt{y^2+1}.
$$

所以：

$$
t=\frac{2y\pm2\sqrt{y^2+1}}2.
$$

### 21.9 第八步：分子提公因数 2，再约分

分子的两项都有因数 $2$：

$$
2y\pm2\sqrt{y^2+1}=2\left(y\pm\sqrt{y^2+1}\right).
$$

于是：

$$
t=\frac{2\left(y\pm\sqrt{y^2+1}\right)}2
=y\pm\sqrt{y^2+1}.
$$

即两个候选值：

$$
t_+=y+\sqrt{y^2+1},\qquad t_-=y-\sqrt{y^2+1}.
$$

### 21.10 第九步：为什么减号解一定要舍掉

换元时已经要求 $t=e^x>0$。又因为：

$$
y^2+1>y^2\quad\Longrightarrow\quad
\sqrt{y^2+1}>\sqrt{y^2}=|y|.
$$

所以：

$$
y-\sqrt{y^2+1}<y-|y|\le0.
$$

减号解总是严格小于零，不符合 $t>0$，必须舍掉。

同时：

$$
y+\sqrt{y^2+1}>y+|y|\ge0.
$$

加号解总是严格大于零，即使 $y$ 为负数也成立，因此可以保留：

$$
\boxed{e^x=y+\sqrt{y^2+1}}.
$$

### 21.11 第十步：两边取自然对数

右边已经证明严格大于零，可以取 $\ln$：

$$
\ln(e^x)=\ln\left(y+\sqrt{y^2+1}\right).
$$

利用 $\ln(e^x)=x$：

$$
\boxed{x=\ln\left(y+\sqrt{y^2+1}\right)}.
$$

这说明每个实数 $y$ 都对应唯一实数 $x$。前面的变形在保留 $t>0$ 后都可逆，因此原函数值域确实为 $\mathbb R$。

### 21.12 第十一步：交换变量名称，写出反函数

把反函数的输入从 $y$ 重新命名为 $x$：

$$
\boxed{f^{-1}(x)=\ln\left(x+\sqrt{x^2+1}\right)},\qquad x\in\mathbb R.
$$

原函数和反函数的定义域、值域都是 $\mathbb R$。反函数对所有实数有定义，因为 $x+\sqrt{x^2+1}>0$ 恒成立。

### 21.13 代回验证

令 $s=\sqrt{y^2+1}$，则 $s^2-y^2=1$，所以：

$$
(s+y)(s-y)=1\quad\Longrightarrow\quad \frac1{s+y}=s-y.
$$

若 $x=\ln(y+s)$，那么 $e^x=y+s$，$e^{-x}=1/(y+s)=s-y$。代回原式：

$$
\frac{e^x-e^{-x}}2
=\frac{(y+s)-(s-y)}2
=\frac{2y}{2}=y.
$$

验证通过。还可以快速检查 $f(0)=0$，反函数在 $0$ 处的值也是 $\ln1=0$。

### 21.14 记住解题主线

$$
\text{负指数改倒数}
\longrightarrow \text{消分母、移项}
\longrightarrow \text{设 }t=e^x>0
\longrightarrow \text{解二次方程}
\longrightarrow \text{舍负根}
\longrightarrow \text{取 }\ln.
$$

先理解每一步使用了哪条基础规则，再尝试独立完成整题。

{{< study-nav >}}
