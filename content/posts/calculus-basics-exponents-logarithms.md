+++
title = '高数基础复习笔记（2）：指数与对数'
date = '2026-09-17T12:00:00+08:00'
lastmod = '2026-09-17T12:00:00+08:00'
draft = false
description = '理解指数运算、常数 e、指数函数与自然对数，特别注意公式成立的条件。'
summary = '理解指数运算、常数 e、指数函数与自然对数，特别注意公式成立的条件。'
tags = ['高等数学', '学习笔记']
isCJKLanguage = true
ShowToc = true
TocOpen = true
UseHugoToc = true
studyseries = 'calculus-basics'
studyorder = 2
+++

理解指数运算、常数 e、指数函数与自然对数，特别注意公式成立的条件。

> 本文是「高数基础复习笔记」系列第 2 篇，对应原笔记第 8—10 节。除非特别说明，只讨论**实数范围**。
>
> [总目录与公式速查]({{< relref "/posts/calculus-basics.md" >}})。保留原章节编号，便于对照复习；本篇会随学习进度持续补充。

## 8. 指数运算 {#section-8}

$a^n$ 中，$a$ 是底数，$n$ 是指数。例如 $2^3=2\times2\times2=8$。

### 8.1 零次方与负指数

$$
a^0=1\quad(a\ne0),\qquad
a^{-n}=\frac1{a^n}\quad(a\ne0,\ n\text{ 为正整数}).
$$

例如 $2^{-3}=1/8$。**负指数表示倒数，不表示结果是负数。** 本笔记不把 $0^0$ 当作可套用零次方规则的式子。

### 8.2 常用运算法则

为使下面公式适用于任意实数指数，统一假设底数 $a>0,b>0$：

$$
a^m a^n=a^{m+n},\qquad
\frac{a^m}{a^n}=a^{m-n},\qquad
(a^m)^n=a^{mn},\qquad
(ab)^m=a^m b^m.
$$

例如：

$$
e^x e^x=e^{2x},\qquad (e^x)^2=e^{2x},\qquad e^x e^{-x}=1.
$$

同底数**相乘**才能把指数相加；$e^x+e^x=2e^x$，一般不等于 $e^{2x}$。

### 8.3 分数指数

对 $a>0$：

$$
a^{1/2}=\sqrt a,\qquad a^{1/3}=\sqrt[3]a,\qquad
a^{m/n}=\sqrt[n]{a^m}\quad(m,n\text{ 为正整数}).
$$

## 9. 常数 e 与指数函数 {#section-9}

$$
e\approx2.71828.
$$

$e$ 是固定的无理数，像 $\pi$ 一样，不是变量。初学时不必背很多位小数。

$$
e^0=1,\qquad e^1=e,\qquad e^{-x}=\frac1{e^x}.
$$

指数函数 $y=e^x$ 的定义域为 $\mathbb R$，值域为 $(0,+\infty)$。它随 $x$ 增大而严格增大，并且：

$$
\boxed{e^x>0\quad\text{对任意实数 }x\text{ 都成立}}
$$

$x<0$ 时也有 $e^x>0$；指数可以是负数，函数值仍然是正数。

## 10. 对数与自然对数 ln {#section-10}

### 10.1 对数是指数的反运算

$$
\boxed{\log_a b=c\quad\Longleftrightarrow\quad a^c=b}
$$

条件是 $a>0,a\ne1,b>0$；$a$ 叫底数，$b$ 叫真数。

例如 $2^3=8$，所以 $\log_2 8=3$。对数问的是：“底数的多少次方等于真数？”

### 10.2 自然对数

$$
\boxed{\ln x=\log_e x}
$$

$\ln x$ 问的是“$e$ 的多少次方等于 $x$”：

$$
\ln e=1,\qquad \ln1=0,\qquad \ln5\approx1.609.
$$

没有要求小数近似时，可以直接保留 $\ln5$。

### 10.3 定义域与值域

$\ln x$ 的定义域是 $(0,+\infty)$，值域是 $\mathbb R$。$\ln0$ 和负数的自然对数在实数范围内没有定义。

$$
\boxed{\ln(e^x)=x\quad(x\in\mathbb R)},\qquad
\boxed{e^{\ln x}=x\quad(x>0)}.
$$

例如 $e^u=5$，两边取自然对数得到 $u=\ln5$。

### 10.4 常用对数规则

当 $u>0,v>0,k\in\mathbb R$ 时：

$$
\ln(uv)=\ln u+\ln v,\qquad
\ln\frac uv=\ln u-\ln v,\qquad
\ln(u^k)=k\ln u.
$$

一般**不能**把 $\ln(u+v)$ 拆成 $\ln u+\ln v$。对于 $u\ne0$，$\ln(u^2)=2\ln|u|$；若写成 $2\ln u$，则需要 $u>0$。

{{< study-nav >}}
