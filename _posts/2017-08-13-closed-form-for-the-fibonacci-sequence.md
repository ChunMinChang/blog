---
layout: post
title: Closed Form for the Fibonacci Sequence
category: Math
tags: [Fibonacci]
mathjax: true
comments: true
---
The _Fibonacci_ number can be defined as:

$$
F_n = F_{n-1} + F_{n-2}
$$

where $$F_0 = 0, F_1 = 1$$.

Assume $$F_n = s \cdot r^n$$,
then $$F_n = F_{n-1} + F_{n-2}$$ can be rewritten into:

$$
s \cdot r^n = s \cdot r^{n-1} + s \cdot r^{n-2}
$$

Thus,

$$
s \cdot r^{n-2} \cdot (r^2 - r - 1) = 0
$$

and $$r = \frac{1\pm \sqrt{5}}{2}$$

Let $$p = \frac{1 + \sqrt{5}}{2}$$, $$r = \frac{1 - \sqrt{5}}{2}$$,
and $$x_k = u \cdot p^k + v \cdot q^k$$,
we now show that $$x_k$$ is also a solution for $$F_k$$.
That is, we need to show $$x_k = x_{k-1} + x_{k-2}$$.

_Proof_:

$$
\begin{align}
x_{k-1} + x_{k-2}
&= u \cdot p^{k-1} + v \cdot q^{k-1} + u \cdot p^{k-2} + v \cdot q^{k-2} \\
&= u \cdot p^{k-2} \cdot (1 + p) + v \cdot q^{k-2} \cdot (1 + q) \\
\end{align}
$$

Since $$r + 1 = r^2(\text{by } r^2 - r - 1 = 0)$$,
we know $$p + 1 = p^2$$ and $$q + 1 = q^2$$
Thus,

$$
\begin{align}
x_{k-1} + x_{k-2}
&= u \cdot p^{k-2} \cdot (1 + p) + v \cdot q^{k-2} \cdot (1 + q) \\
&= u \cdot p^{k-2} \cdot p^2 + v \cdot q^{k-2} \cdot q^2 \\
&= u \cdot p^k + v \cdot q^k \\
&= x_k
\end{align}
$$

Next, if we take $$x_k$$ as our closed form for the _Fibonacci_ number,
we also need to know what $$u, v$$ are.
Fortunately, by the definition, we already know $$F_0 = 0, F_1 = 1$$,
so

$$
\begin{align}
x_0 &= 0 = u \cdot p^0+ v \cdot q^0 = u + v \\
x_1 &= 1 = u \cdot p + v \cdot q
= u \cdot \frac{1 + \sqrt{5}}{2} + v \cdot \frac{1 - \sqrt{5}}{2} \\
\Rightarrow 2 &= u + u \cdot \sqrt{5} + v - v \cdot \sqrt{5} \\
&= (u + v) + \sqrt{5} \cdot (u - v) \\
&= \sqrt{5} \cdot (u - v)
\end{align}
$$

By $$u + v = 0$$ and $$u - v = \frac{2}{\sqrt{5}}$$, we know that

$$
\begin{align}
u &= \frac{1}{\sqrt{5}} \\
v &= \frac{-1}{\sqrt{5}}
\end{align}
$$

As a result,

$$
\begin{align}
x_k &= u \cdot p^k + v \cdot q^k \\
&= \frac{1}{\sqrt{5}} \cdot (\frac{1 + \sqrt{5}}{2})^k - \frac{1}{\sqrt{5}} \cdot (\frac{1 - \sqrt{5}}{2})^k \\
&= \frac{1}{\sqrt{5}} \cdot [ (\frac{1 + \sqrt{5}}{2})^k -  (\frac{1 - \sqrt{5}}{2})^k ]
\end{align}
$$


## References
- http://math.ucr.edu/~res/math153/history07a.pdf
- http://gozips.uakron.edu/~crm23/fibonacci/fibonacci.htm
