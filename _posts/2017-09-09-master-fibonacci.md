---
layout: post
title: Master Fibonacci
category: [Algorithm]
tags: [Fibonacci, Recursion, Dynamic Programming]
mathjax: true
comments: true
---
The _Fibonacci_ number is defined as:

$$
F_n = F_{n-1} + F_{n-2}
$$

where $$F_0 = 0, F_1 = 1$$.

It can be directly written into the following most common code
when we learned what the recursion is:
```cpp
///////////////////////////////////////////////////////////////////////////////
// Recursive: O(2^n)
uint64_t fibonacci(unsigned int n)
{
  return (n <= 1) ? n : fibonacci(n-1) + fibonacci(n-2);
}
```
However, if you try calculating $$F_{100}$$,
then you will wait a long long time to get the result
since it has so many overlapping processes.
For example, if we calculate $$F_4$$,
then there are duplicated calculations(overlapping substructures) for $$F_2$$:

$$
\begin{matrix}
 & & & & & & & 4 & & & & & \\
 & & & & & & \diagup & & \diagdown & & & & \\
 & & & & & \diagup & & & & \diagdown & & & \\
 & & & & 3 & & & & & & 2 & & \\
 & & & \diagup & & \diagdown & & & & \diagup & & \diagdown & \\
 & & 2 & & & & 1 & & 1 & & & & 0 \\
 & \diagup & & \diagdown & & & & & & & & & \\
 1 & & & & 0 & & & & & & & &
\end{matrix}
$$

The larger $$n$$ is, the more overlapping processes we have.
As a result, the time-complexity is $$O(2^n)$$.

### Memoization

To avoid that, we can use a __cache__ to save all the results
and check it first before any calculation,
so all the $$F_k$$ we need, for $$k \in [0, n]$$,
will be computed just once.
Therefore, the time complexity can be shorten to $$O(n)$$.

```cpp
///////////////////////////////////////////////////////////////////////////////
// Recursive with memoization: O(n)
std::vector<uint64_t> mem = { 0, 1 }; // F(k) = mem[k], F(0) = 0, F(1) = 1.
uint64_t fibonacci(unsigned int n)
{
  if (n + 1 > mem.size()) { // if n is not calculated yet
    mem.push_back(fibonacci(n-1) + fibonacci(n-2));
  }
  return mem[n];
}
```

### Dynamic programming

The above implementation needs extra space to save the results,
and pay time for memory allocation.
If we iteratively calculate $$F_n$$
from $$F_0, F_1$$ to $$F_2$$, $$F_3$$, ... then we can get $$F_n$$
without extra memory:

The above implementation needs extra space to save the results,
and pay time for memory allocation.
If we iteratively calculate $$F_n$$
from $$F_0, F_1$$ to $$F_2$$, $$F_3$$, ...,
to $$F_{n-1}$$, $$F_n$$ or $$F_n$$, $$F_{n+1}$$
then we can use only three or four variables to get $$F_n$$:

```cpp
///////////////////////////////////////////////////////////////////////////////
// Dynamic programming: O(n)
uint64_t fibonacci(unsigned int n)
{
  uint64_t a = 0, b = 1; // a = F(k), b = F(k+1), k = 0 now.
  for (unsigned int k = 1 ; k <= n ; ++k) { // loop k from 1 to n.
    std::swap(a, b); // a = F(k+1), b = F(k)
    b += a; // b = F(k) + F(k+1) = F(k+2)
  }
  return a;
}
```

or

```cpp
///////////////////////////////////////////////////////////////////////////////
// Dynamic programming: O(n)
uint64_t fibonacci(unsigned int n)
{
  uint64_t a = 0, b = 1, sum = 0; // a = F(0), b = F(1)
  for (unsigned int i = 1 ; i < n ; ++i) { // run if n >= 2
    sum = a + b; // sum = F(i+1)
    a = b;       // a = F(i)
    b = sum;     // b = F(i+1)
  }
  // Now, i = n, sum = F(n), a = F(n-1), b = F(n)
  return (n < 2) ? n : sum;
}
```

They also run in $$O(n)$$ with less memory consumption
than _memoization_ approach.
Furthermore, they avoid the memory overhead for the _activation records_
on the _stack segment/space_ for the recursions.
(The recursion will call itself multiple times,
so it will push multiple _activation records_ for the same function itself,
with different arguments, into the _stack segment/space_
of the process loading the program.)

### Closed-form

In fact, the _Fibonacci_ number can be calculated by the following formula:

$$
F_n = \frac{1}{\sqrt{5}} \cdot [ (\frac{1 + \sqrt{5}}{2})^n -  (\frac{1 - \sqrt{5}}{2})^n ]
$$

(Please read
<!-- [this post]({% post_url 2017-08-13-closed-form-for-the-fibonacci-sequence %}) -->
[this post](https://chunminchang.github.io/blog/post/closed-form-for-the-fibonacci-sequence)
to know how it's derived.)

```cpp
///////////////////////////////////////////////////////////////////////////////
// closed-form: O(log(n))
//   Theoretically, the power of n could be done in O(log(n)), but it's
//   complicated to calculate the floating numbers.
uint64_t fibonacci(unsigned int n)
{
  // double sqrt5 = sqrt((double)5);
  double sqrt5 = 2.2360679775;
  return (pow((1 + sqrt5) / 2, n) - pow((1 - sqrt5) / 2, n)) / sqrt5;
}
```

Its time-complexity depends on how the power of $$n$$ is calculated.
It could be done in $$O(\log n)$$ time(we will explain it below).
However, the floating point operations limit the calculable number of $$n$$,
and it might block the performance.


### Matrix Algebra

The _Fibonacci_ numbers can be written into the following matrix:

$$
\vec{F_n} =
\begin{bmatrix} F_n \\ F_{F - 1} \end{bmatrix}
=
\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}
\cdot
\begin{bmatrix} F_{n - 1} \\ F_{n - 2} \end{bmatrix}
$$

, so it could be easily expanded by the same rule:

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
\vdots
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

(Please read
<!-- [this post]({% post_url 2017-08-22-matrix-difference-equation-for-fibonacci-sequence %}) -->
[this post](https://chunminchang.github.io/blog/post/matrix-difference-equation-for-fibonacci-sequence)
for more discussion.)

That is, the _Fibonacci_ matrix turns into a perfect power.
Applying
<!-- [exponentiation by squaring]({% post_url 2017-08-19-exponentiation-by-squaring %}) -->
[exponentiation by squaring](https://chunminchang.github.io/blog/post/exponentiation-by-squaring)
:

$$
k^n =
\begin{cases}
(k^2)^\frac{n}{2},  & \text{if $n$ is even} \\
k \cdot (k^2)^\frac{n-1}{2}, & \text{if $n$ is odd}
\end{cases}
$$

, we could implement the above idea to:

```cpp
///////////////////////////////////////////////////////////////////////////////
// Power by matrix exponentiation: O(log(n))
// Matrix A:
//  <---  cols: n  --->
// +-                 -+
// | A11, A12, ... A1n |   ^
// | A21, A22, ... A2n |   |
// | ...               | rows: m
// | ...               |   |
// | Am1, Am2, ... Amn |   v
// +-                 -+
class Matrix
{
public:
  Matrix(unsigned int r, unsigned int c,
         std::vector<std::vector<uint64_t>> d)
    : rows(r)
    , cols(c)
    , data(d)
  {
  }

  Matrix(unsigned int r, unsigned int c)
    : rows(r)
    , cols(c)
  {
    assert(rows && cols);
    data.resize(rows);
    for (unsigned int i = 0 ; i < rows ; ++i) {
      data[i].resize(cols);
    }
  }

  ~Matrix()
  {
  }

  uint64_t Read(unsigned int r, unsigned int c)
  {
    return data[r][c];
  }

  // friend std::ostream& operator<<(std::ostream& os, const Matrix& m)
  // {
  //   for (unsigned int i = 0; i < m.rows; ++i) {
  //     for (unsigned int j = 0; j < m.cols; ++j) {
  //       os << m.data[i][j] << " ";
  //     }
  //     os << std::endl;
  //   }
  //   return os;
  // }

  Matrix operator*(const Matrix& other)
  {
    assert(cols == other.rows); // Check if they can be multiplied.

    Matrix z(rows, other.cols);
    for (unsigned int i = 0 ; i < rows ; ++i) {
      for (unsigned int j = 0 ; j < other.cols; ++j) {
        for (unsigned int k = 0 ; k < cols; ++k) {
          z.data[i][j] += data[i][k] * other.data[k][j];
        }
      }
    }

    return z;
  }

  // Calculate the power by fast doubling:
  //   k ^ n = (k^2) ^ (n/2)          , if n is even
  //        or k * (k^2) ^ ((k-1)/2)  , if n is odd
  Matrix pow(unsigned int n)
  {
    Matrix k(*this); // Copy constructor = Matrix x(rows, cols, data);
    Matrix r = Identity(rows);
    while (n) {
      if (/*n % 2*/n & 1) {
        r = r * k;
      }
      k = k * k;
      /*n /= 2*/n >>= 1;
    }
    return r;
  }

private:
  Matrix Identity(unsigned int size)
  {
    Matrix z(size, size);
    for (unsigned int i = 0 ; i < size ; ++i) {
      z.data[i][i] = 1;
    }
    return z;
  }

  unsigned int rows;
  unsigned int cols;
  std::vector<std::vector<uint64_t>> data;
};

// The Fibonacci matrix can be written into the following equation:
// +-             -+   +-    -+^n
// | F(n+1)   F(n) |   | 1  1 |
// |               | = |      |
// | F(n)   F(n-1) |   | 1  0 |
// +-             -+   +-    -+
uint64_t fibonacci(unsigned int n)
{
  Matrix F { 2, 2, {
    { 1, 1 },
    { 1, 0 }
  } };

  // Using F.data[0][1] since n might be 0.
  // (we need to power by n - 1 if we return F.data[0][0].)
  F = F.pow(n);
  return F.Read(0, 1);
}
```

Its time-complexity is $$O(\log n)$$ by halving and halving.
Without the floating point operations,
the $$n$$ could be larger than using the _closed-form_ approach.

To make it faster, you can use native array instead of ```std::vector```,
but you need to manage the memory usage by yourself.
Please read
<!-- [this post]({% post_url 2017-08-22-matrix-difference-equation-for-fibonacci-sequence %}) -->
[this post](https://chunminchang.github.io/blog/post/matrix-difference-equation-for-fibonacci-sequence)
to know how to do it.

### Fast doubling

The following equations:

$$
\begin{align}
F_{2n+1} &= {F_{n+1}}^2 + {F_n}^2
\\
F_{2n} &= F_n \cdot (F_{n+1} + F_{n-1}) \\
       &= F_n \cdot (F_{n+1} + (F_{n+1} - F_n)) \\
       &= F_n \cdot (2 \cdot F_{n+1} - F_n)
\end{align}
$$

can be derived by applying $$2n$$ to the above _Fibonacci_ matrix:

$$
\begin{align}
\begin{bmatrix} F_{2n+1} & F_{2n} \\ F_{2n} & F_{2n - 1} \end{bmatrix}
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

Hence, we could calculate $$F_n$$ by:

$$
F_n =
\begin{cases}
F_{2n'},  & \text{if $n$ is even} \\
F_{2n'+1}, & \text{if $n$ is odd}
\end{cases}
$$

As a consequence, we could use $$F_{n'}, F_{n' + 1}$$
to compute $$F_n$$ by the following program:
(Please read
<!-- [this post]({% post_url 2017-08-31-calculating-fibonacci-numbers-by-fast-doubling %}) -->
[this post](https://chunminchang.github.io/blog/post/calculating-fibonacci-numbers-by-fast-doubling)
to know how the code is derived.)

```cpp
uint64_t fibonacci(unsigned int n)
{
  // The position of the highest bit of n.
  // So we need to loop `h` times to get the answer.
  // Example: n = (Dec)50 = (Bin)00110010, then h = 6.
  //                               ^ 6th bit from right side
  unsigned int h = 0;
  for (unsigned int i = n ; i ; ++h, i >>= 1);

  uint64_t a = 0; // F(0) = 0
  uint64_t b = 1; // F(1) = 1
  // There is only one `1` in the bits of `mask`. The `1`'s position is same as
  // the highest bit of n(mask = 2^(h-1) at first), and it will be shifted right
  // iteratively to do `AND` operation with `n` to check `n / 2^j` is odd
  // or even.
  for (unsigned int mask = 1 << (h - 1) ; mask ; mask >>= 1) { // Run h times!
    // Let j = h-i (looping from i = 1 to i = h),
    // n_j = floor(n / 2^j) = n >> j (n_j = n when j = 0), k = floor(n_j / 2),
    // then a = F(k), b = F(k+1) now.
    uint64_t c = a * (2 * b - a); // F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    uint64_t d = a * a + b * b;   // F(2k+1) = F(k)^2 + F(k+1)^2

    if (mask & n) { // n_j is odd: k = (n_j-1)/2 => n_j = 2k + 1
      a = d;        //   F(n_j) = F(2k + 1)
      b = c + d;    //   F(n_j + 1) = F(2k + 2) = F(2k) + F(2k + 1)
    } else {        // n_j is even: k = n_j/2 => n_j = 2k
      a = c;        //   F(n_j) = F(2k)
      b = d;        //   F(n_j + 1) = F(2k + 1)
    }
  }

  return a;
}
```

Its time-complexity is also $$O(\log n)$$ by halving and halving.
In contrast to _matrix algebra_ approach,
there is no need for using matrix
that contains the duplicated $$F_k$$,
so it will be faster.


### Performance

| Approach            | $$F_{45}$$ | $$F_{13100}$$ | $$F_{13500}$$ | $$F_{29108}$$ |
| ------------------- | ---------- | ------------- | ------------- | ------------- |
| Recursive           | 7440.61    |               |               |               |
| Memoization         | 0.034841   | 3.03045       | 3.05931       | 6.10806       |
| Dynamic programming | 0.000508   | 0.052462      | 0.05395       | 0.1069        |
| Closed-form         | 0.030075   |               |               |               |
| Matrix Algebra      | 0.02013    | 0.052985      | 0.052427      | 0.050423      |
| Fast doubling       | 0.000446   | 0.000737      | 0.000785      | 0.000724      |

The above results are the time in _millisecond_ for calculating $$F_n$$.
It will take too long time to get the results from the _recursive_ approach,
so we skip it.
The _closed-form_ approach is also ignored
since the floating point operations only work
when $$n \leq 97$$ in above implementation.

### Conclusion

Although the performance is platform-dependent,
it still indicates that:

- The _fast doubling_ approach is always the fastest way
  and its performance is far far better than others.
- The _dynamic programming_ approach is faster than _matrix algebra_ one
  when $$n$$ is small ($$n \leq 13000$$ here),
  but slower when $$n$$ is large.
- Therefore, if you are pretty sure you have a small $$n$$,
  and the bottleneck of your algorithm doesn't depend on
  the _Fibonacci_ calculation, then _dynamic programming_ is acceptable
  and it's easier to implement.


This post is the end of my journey for the _Fibonacci_ calculation.
Hope you enjoyed.
All the above code are uploaded to [gist here][gist].
Please clone them to play with it.

I will start another journey for other interesting topics soon.
Stay tuned!

[gist]: https://gist.github.com/ChunMinChang/b6325c148e8aff15b6e72dcac0aa904e "Ways to calculate Fibonacci"
