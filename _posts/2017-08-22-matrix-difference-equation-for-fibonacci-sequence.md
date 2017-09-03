---
layout: post
title: Matrix Difference Equation for Fibonacci Sequence
category: [Algorithm, Math]
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

Since $$S$$ can transit the state from $$\vec{x_{n-1}}$$ to $$\vec{x_n}$$,
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
      \cdots +
      c_0 \cdot y_0
$$


, where $$c_k$$ is constant and $$k \in [0, n-1] $$ is a integer.

then we can rewritten the equations into:

$$
\vec{y_n} =
\begin{bmatrix}
  y_n \\
  y_{n - 1} \\
  \vdots \\
  y_1
\end{bmatrix}
=
\begin{bmatrix}
  c_{n - 1} & c_{n - 2} & \cdots & c_1 & c_0 \\
  1 & 0 & \cdots & 0 & 0 \\
  0 & 1 & \cdots & 0 & 0 \\
  \vdots \\
  0 & 0 & \cdots & 1 & 0
\end{bmatrix}
\cdot
\begin{bmatrix}
  y_{n - 1} \\
  y_{n - 2} \\
  \vdots \\
  y_0
\end{bmatrix}
= C \cdot\ \vec{y_{n - 1}}
$$

and we could expand the matrix to

$$
\begin{bmatrix}
  y_{2n - 1} & \cdots & y_{n + 1} & y_n \\
  y_{2n - 2} & \cdots & y_n & y_{n - 1} \\
  \vdots \\
  y_n & \cdots & y_2 & y_1
\end{bmatrix}
=
\begin{bmatrix}
  c_{n - 1} & c_{n - 2} & \cdots & c_1 & c_0 \\
  1 & 0 & \cdots & 0 & 0 \\
  0 & 1 & \cdots & 0 & 0 \\
  \vdots \\
  0 & 0 & \cdots & 1 & 0
\end{bmatrix}
\cdot
\begin{bmatrix}
  y_{2n - 2} & \cdots & y_n & y_{n - 1} \\
  y_{2n - 3} & \cdots & y_{n - 1} & y_{n - 2} \\
  \vdots \\
  y_{n - 1} & \cdots & y_1 & y_0
\end{bmatrix}
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

#### Computing _Fibonacci_ number by exponentiation
By the above formula, the _Fibonacci_ number $$F_n$$
can be calculated in $$O(\log n)$$.
The key is to compute the exponentiation by squaring.

$$
k^n =
\begin{cases}
(k^2)^\frac{n}{2},  & \text{if $n$ is even} \\
k \cdot (k^2)^\frac{n-1}{2}, & \text{if $n$ is odd}
\end{cases}
$$

I explained how to do it in my [previous post]({% post_url 2017-08-19-exponentiation-by-squaring %}).
Please read it if you need.

As a result, we can compute _Fibonacci_ number $$F_n$$ as fellow:

##### By ```std::vector```
```cpp
class Matrix
{
public:
  Matrix(unsigned int r, unsigned int c,
         std::vector<std::vector<unsigned int>> d)
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

  unsigned int Read(unsigned int r, unsigned int c)
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
  //        or k x (k^2) ^ ((k-1)/2)  , if n is odd
  Matrix pow(unsigned int n)
  {
    Matrix x(*this); // Copy constructor = Matrix x(rows, cols, data);
    Matrix r = Identity(rows);
    while (n) {
      if (/*n % 2*/n & 1) {
        r = r * x;
      }
      x = x * x;
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
  std::vector<std::vector<unsigned int>> data;
};

// The Fibonacci matrix can be written into the following equation:
// +-             -+   +-    -+^n
// | F(n+1)   F(n) |   | 1  1 |
// |               | = |      |
// | F(n)   F(n-1) |   | 1  0 |
// +-             -+   +-    -+
uint64_t fibonacci_matrix(unsigned int n)
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

##### By Native Array
```cpp
class Matrix
{
public:
  Matrix(unsigned int r, unsigned int c, unsigned int** d = nullptr)
    : rows(r)
    , cols(c)
    , data(d)
  {
    if (!data) {
      AllocateData();
    }
  }

  Matrix(const Matrix& other) // copy ctor
    : rows(other.rows)
    , cols(other.cols)
  {
    assert(!data);
    AllocateData();
    for (unsigned int i = 0 ; i < rows ; ++i) {
      for (unsigned int j = 0 ; j < cols ; ++j) {
        data[i][j] = other.data[i][j];
      }
    }
  }

  ~Matrix()
  {
    FreeData();
  }

  unsigned int Read(unsigned int r, unsigned int c)
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

  Matrix& operator=(Matrix&& other) noexcept // move assignment
  {
    if(this != &other) {
      FreeData(); // Free this data if it exists.
      // Move original other.data to data and set other.data to nullptr.
      // data = std::exchange(other.data, nullptr); // C++14
      data = other.data;
      other.data = nullptr;
    }
    return *this;
  }

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
  //        or k x (k^2) ^ ((k-1)/2)  , if n is odd
  Matrix pow(unsigned int n)
  {
    Matrix x(*this); // Copy constructor = Matrix x(rows, cols, data);
    Matrix r = Identity(rows);
    while (n) {
      if (/*n % 2*/n & 1) {
        r = r * x;
      }
      x = x * x;
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

  void AllocateData()
  {
    assert(!data);
    data = (unsigned int**) calloc(rows, sizeof(unsigned int*));
    for (unsigned int i = 0 ; i < rows ; ++i) {
      data[i] = (unsigned int*) calloc(cols, sizeof(unsigned int));
    }
  }

  void FreeData()
  {
    if (!data) {
      return;
    }

    assert(rows);
    for (unsigned int i = 0 ; i < rows ; ++i) {
      free(data[i]);
    }
    free(data);
    data = nullptr;
  }

  unsigned int rows;
  unsigned int cols;
  unsigned int** data = nullptr;
};

// The Fibonacci matrix can be written into the following equation:
// +-             -+   +-    -+^n
// | F(n+1)   F(n) |   | 1  1 |
// |               | = |      |
// | F(n)   F(n-1) |   | 1  0 |
// +-             -+   +-    -+
uint64_t fibonacci_matrix(unsigned int n)
{
  Matrix F { 2, 2, new unsigned int*[2] {
    new unsigned int[2] { 1, 1 },
    new unsigned int[3] { 1, 0 }
  } };

  // Using F.data[0][1] since n might be 0.
  // (we need to power by n - 1 if we return F.data[0][0].)
  F = F.pow(n);
  return F.Read(0, 1);
}
```

### Fast Doubling

If we calculate $$F_{2n}$$ directly, we can get the equation as fellow:

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

This two equations help us to calculate $$F_{N}$$ in $$O(\log N)$$ time,
since $$F_{N}$$ can be derived from $$F_{2N'}$$ or $$F_{2N' + 1}$$.

#### implementation

```cpp
///////////////////////////////////////////////////////////////////////////////
// Fast doubling: O(log(n))
//   Using 2n to the Fibonacci matrix above, we can derive that:
//     F(2n)   = F(n) * [ 2 * F(n+1) – F(n) ]
//     F(2n+1) = F(n+1)^2 + F(n)^2
//     (and F(2n-1) = F(n)^2 + F(n-1)^2)
uint64_t fib(unsigned int n)
{
  // When n = 2: k = 1 and we want to use F(k+1) to calculate F(2k),
  // However, F(2k) = F(k+1) = F(2) is unknown then.
  if (n < 2) {
    return n; // F(0) = 0, F(1) = 1.
  } else if (n == 2) {
    return 1; // F(2) = 1
  }

  unsigned int k = 0;
  if (n % 2) { // By F(n) = F(2k+1) = F(k+1)^2 + F(k)^2
    k = (n - 1) / 2;
    return fib(k + 1) * fib(k + 1) + fib(k) * fib(k);
  } else { // By F(n) = F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    k = n / 2;
    return fib(k) * (2 * fib(k + 1) - fib(k));
  }
}
```
The implementation are easy to understand
but it still has a lot of room to improve.
We will discuss it in [next post]({% post_url 2017-08-31-calculating-fibonacci-numbers-by-fast-doubling %}).
Stay tuned!

[rr]: https://en.wikipedia.org/wiki/Recurrence_relation "Recurrence relation"
[mde]: https://en.wikipedia.org/wiki/Matrix_difference_equation "Matrix difference equation"
