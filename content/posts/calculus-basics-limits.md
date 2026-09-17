+++
title = '高数基础复习笔记（5）：极限入门'
date = '2026-09-17T12:00:00+08:00'
lastmod = '2026-09-17T12:00:00+08:00'
draft = false
description = '区分函数值与极限，理解左右极限，并认识几个常见极限。'
summary = '区分函数值与极限，理解左右极限，并认识几个常见极限。'
tags = ['高等数学', '学习笔记']
isCJKLanguage = true
ShowToc = true
TocOpen = true
UseHugoToc = true
studyseries = 'calculus-basics'
studyorder = 5
+++

区分函数值与极限，理解左右极限，并认识几个常见极限。

> 本文是「高数基础复习笔记」系列第 5 篇，对应原笔记第 20 节。除非特别说明，只讨论**实数范围**。
>
> [总目录与公式速查]({{< relref "/posts/calculus-basics.md" >}})。保留原章节编号，便于对照复习；本篇会随学习进度持续补充。

## 20. 极限入门 {#section-20}

### 20.1 lim 的意思

$$
\lim_{x\to a}f(x)=L
$$

表示：当 $x$ 越来越接近 $a$ 时，$f(x)$ 越来越接近 $L$。求极限关注的是附近的变化，不要求 $x$ 真的等于 $a$。

例如：

$$
\lim_{x\to2}(x+3)=5.
$$

### 20.2 极限不一定等于代入值

$$
\lim_{x\to1}\frac{x^2-1}{x-1}
=\lim_{x\to1}(x+1)=2.
$$

原式在 $x=1$ 处没有定义，但对附近的 $x\ne1$ 可约分，所以极限存在。这里的 $0/0$ 不是一个数，也不是最终答案。

函数在某点连续时，该点极限等于函数值，可以直接代入；不是所有题都满足这个条件。

### 20.3 左极限与右极限

$$
\lim_{x\to a^-}f(x),\qquad \lim_{x\to a^+}f(x)
$$

分别表示从左侧、右侧接近 $a$。在两侧都有定义的情形，双侧有限极限存在，当且仅当左右极限存在且相等。

[第 17 节的分段函数]({{< relref "/posts/calculus-basics-function-relations.md#section-17" >}})满足：

$$
\lim_{x\to1^-}f(x)=1,\qquad \lim_{x\to1^+}f(x)=2.
$$

因此 $x\to1$ 时的双侧极限不存在，虽然 $f(1)=2$ 是有定义的。

### 20.4 常见极限

$$
\lim_{x\to0}\frac{\sin x}{x}=1\qquad\text{（}x\text{ 用弧度）}.
$$

这不是说 $\sin x=x$ 对所有 $x$ 成立，而是比值在 $x\to0$ 时趋近于 $1$。

$$
\lim_{x\to+\infty}\frac1x=0,\qquad
\lim_{x\to-\infty}e^x=0.
$$

“趋近于 $0$”不表示一定取到 $0$：$e^x$ 永远大于零。

{{< study-nav >}}
