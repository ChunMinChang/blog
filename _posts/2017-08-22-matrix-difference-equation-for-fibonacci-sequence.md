---
layout: post
title: Matrix Difference Equation for Fibonacci Sequence
category: [Math]
tags: [Fibonacci]
mathjax: true
comments: true
---

## [Recurrence relation][rr]

Suppose we have a difference equation defined by:

$$
x_n = a \cdot x_{n - 1} + b \cdot x_{n - 2}
$$

Then,

$$
\begin{align}
x_n &= a \cdot x_{n - 1} + b \cdot x_{n - 2}
\\
x_{n - 1} &= 1 \cdot x_{n - 1} + 0 \cdot x_{n - 2}
\end{align}
$$

could be written into a matrix form([_matrix difference equation_][mde]):

$$
\vec{x_n} =
\begin{bmatrix} x_n \\ x_{n - 1} \end{bmatrix}
=
\begin{bmatrix} a & b \\ 1 & 0 \end{bmatrix}
\cdot
\begin{bmatrix} x_{n - 1} \\ x_{n - 2} \end{bmatrix}
=
S \cdot \vec{x_{n-1}}
$$

Since $$S$$ can transit the state from \vec{x_{n-1}} to \vec{x_n},
the state vector can be expanded by adding any pair $$\vec{x_{t-1}}, \vec{x_t}$$
to the above equation.

For example,

$$
\begin{align}
x_t, x_n &= a \cdot x_{t - 1} + b \cdot x_{t - 2}, a \cdot x_{n - 1} + b \cdot x_{n - 2}
\\
x_{t - 1}, x_{n - 1} &= 1 \cdot x_{t - 1} + 0 \cdot x_{t - 2}, 1 \cdot x_{n - 1} + 0 \cdot x_{n - 2}
\end{align}
$$

can be written into

$$
\begin{bmatrix} x_t & x_n \\ x_{t - 1} & x_{n - 1} \end{bmatrix}
=
\begin{bmatrix} a & b \\ 1 & 0 \end{bmatrix}
\cdot
\begin{bmatrix} x_{t - 1} & x_{n - 1} \\ x_{t - 2} & x_{n - 2} \end{bmatrix}
$$


In fact, this can be generalized.
Given $$y_n$$ by:

$$
y_n = c_{n - 1} \cdot y_{n - 1} +
      c_{n - 2} \cdot y_{n - 2} +
      ... +
      c_0 \cdot y_0
$$

then we can rewritten the equations into:

$$
\vec{y_n} =
\begin{bmatrix}
  y_n \\
  y_{n - 1} \\
  ... \\
  y_1
\end{bmatrix}
=
\begin{bmatrix}
  c_{n - 1} & c_{n - 2} & ... & c_1 & c_0 \\
  1 & 0 & ... & 0 & 0 \\
  0 & 1 & ... & 0 & 0 \\
  ... \\
  0 & 0 & ... & 1 & 0
\end{bmatrix}
\cdot
\begin{bmatrix}
  y_{n - 1} \\
  y_{n - 2} \\
  ... \\
  y_0
\end{bmatrix}
= C \cdot\ \vec{y_{n - 1}}
$$

## Fibonacci Sequence

Fibonacci number is defined by:

$$
F_n = F_{n - 1} + F_{n - 2}, \text{where } F_0 = 0 \text{ and } F_1 = 0
$$

Obviously, _Fibonacci_ sequence is a [_difference equation_][rr]
($$a = b = 1$$ in above example) and it could be written in:

$$
\vec{F_n} =
\begin{bmatrix} F_n \\ F_{F - 1} \end{bmatrix}
=
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}
\cdot
\begin{bmatrix} F_{n - 1} \\ F_{n - 2} \end{bmatrix}
=
S \cdot \vec{F_{n-1}}
$$

### Matrix Form

If we expand the $$\vec{F_n}$$ by taking $$t = n + 1$$ in above example,
then

$$
\begin{align}
\begin{bmatrix} F_{n+1} & F_n \\ F_n & F_{n - 1} \end{bmatrix}
&=
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}
\cdot
\begin{bmatrix} F_n & F_{n - 1} \\ F_{n - 1} & F_{n - 2} \end{bmatrix}
\\
&=
{\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^2
\cdot
\begin{bmatrix} F_{n - 1} & F_{n - 2} \\ F_{n - 2} & F_{n - 3} \end{bmatrix}
\\
&= ...
\\
&= {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^{n - 1}
\cdot
\begin{bmatrix} F_2 & F_1 \\ F_1 & F_0 \end{bmatrix}
\\
&= {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^{n - 1}
\cdot
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}
\\
&= {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^n
\end{align}
$$

### Fast Doubling

$$
\begin{align}
\begin{bmatrix} F_{2n+1} & F_2n \\ F_n & F_{2n - 1} \end{bmatrix}
&= {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^{2n}
\\
&= {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^n
\cdot
{\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}}^n
\\
&= \begin{bmatrix} F_{n+1} & F_n \\ F_n & F_{n - 1} \end{bmatrix}
\cdot \begin{bmatrix} F_{n+1} & F_n \\ F_n & F_{n - 1} \end{bmatrix}
\\
&=
\begin{bmatrix}
  {F_{n+1}}^2 + {F_n}^2 & F_n \cdot (F_{n+1} + F_{n-1}) \\
  F_n \cdot (F_{n+1} + F_{n-1}) & {F_n}^2 + {F_{n-1}}^2
\end{bmatrix}
\end{align}
$$

Thus, $$F_{2n}, F_{2n + 1}$$ can be calculated by $$F_n, F_{n+1}$$ since

$$
\begin{align}
F_{2n+1} &= {F_{n+1}}^2 + {F_n}^2
\\
F_{2n} &= F_n \cdot (F_{n+1} + F_{n-1}) \\
       &= F_n \cdot (F_{n+1} + (F_{n+1} - F_n)) \\
       &= F_n \cdot (2 \cdot F_{n+1} - F_n)
\end{align}
$$



[rr]: https://en.wikipedia.org/wiki/Recurrence_relation "Recurrence relation"
[mde]: https://en.wikipedia.org/wiki/Matrix_difference_equation "Matrix difference equation"
