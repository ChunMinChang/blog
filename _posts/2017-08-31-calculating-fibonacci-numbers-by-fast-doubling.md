---
layout: post
title: Calculating Fibonacci Numbers by Fast Doubling
category: [Algorithm, Math]
tags: [Fibonacci]
mathjax: true
comments: true
---
In [previous post]({% post_url 2017-08-22-matrix-difference-equation-for-fibonacci-sequence %}),
we learned how to calculate _Fibonacci_ numbers by _Fast Doubling_ in math.
Today, we will apply it in programming and optimize it step by step.

## Fast Doubling

$$
\begin{align}
F_{2n+1} &= {F_{n+1}}^2 + {F_n}^2
\\
F_{2n} &= F_n \cdot (F_{n+1} + F_{n-1}) \\
       &= F_n \cdot (F_{n+1} + (F_{n+1} - F_n)) \\
       &= F_n \cdot (2 \cdot F_{n+1} - F_n)
\end{align}
$$

It's natural to write a recursive implementation by the above definition.
In the following steps, we will implement recursive versions first,
then try converting it into iterative versions.

### Recursive

Given a $$n$$, we could calculate _Fibonacci_ numbers $$F_n$$ by:

```cpp
if (n % 2) { // n is odd: F(n) = F(((n-1)/2) + 1)^2 + F((n-1) / 2)^2
  unsigned int k = (n - 1) / 2;
  return fib(k) * fib(k) + fib(k + 1) * fib(k + 1);
} else { // n is even: F(n) = F(n/2) * [ 2 * F(n/2 + 1) - F(n/2) ]
  unsigned int k = n / 2;
  return fib(k) * [ 2 * fib(k + 1) - fib(k) ];
}
```

From above code, we can know that the code stack will be entered again and again,
so we need to define when to stop it.

```cpp
if (n == 0) {
  return 0; // F(0) = 0.
} else if (n <= 2) {
  return 1; // F(1) = F(2) = 0.
} else {
  // Keep call itself recursively to get the answer.
  // Put the main body here.
}
```

We only calculate _Fibonacci_ numbers from $$0$$,
so we need to stop when $$n = 0$$.
The ```fib(0)``` may be asked from calculating ```fib(1) = fib(0)*fib(0) + fib(1)*fib(1)```
(by setting $$n = 0$$ to $$F_{2n+1} = {F_{n+1}}^2 + {F_n}^2$$,
so we also need to define ```fib(1) = 1``` directly,
or it will cause an endless recursive loop.

Similarly, the ```fib(1)``` may be asked from calculating ```fib(2) = fib(1) * [2 * fib(2) - fib(1)]```
(by setting $$n = 1$$ to $$F_{2n} = F_n \cdot (2 \cdot F_{n+1} - F_n)$$,
so ```fib(2) = 1``` also needs to be returned directly.

As the result, the code can be written into:
```cpp
///////////////////////////////////////////////////////////////////////////////
// Fast doubling: O(log(n))
//   Using 2n to the Fibonacci matrix above, we can derive that:
//     F(2n)   = F(n) * [ 2 * F(n+1) – F(n) ]
//     F(2n+1) = F(n+1)^2 + F(n)^2
//     (and F(2n-1) = F(n)^2 + F(n-1)^2)
uint64_t fib(unsigned int n)
{
  if (n == 0) {
    return 0; // F(0) = 0.
  } else if (n <= 2) {
    return 1; // F(1) = F(2) = 0.
  }

  unsigned int k = 0;
  if (n % 2) { // By F(n) = F(2k+1) = F(k+1)^2 + F(k)^2
    k = (n - 1) / 2;
    return fib(k) * fib(k) + fib(k + 1) * fib(k + 1);
  } else { // By F(n) = F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    k = n / 2;
    return fib(k) * (2 * fib(k + 1) - fib(k));
  }
}
```

Now, let we look where we could improve from this simple version.
We use duplicated ```fib(k)``` and ```fib(k + 1)``` to calculate ```fib(n)```.
That is, we will have two duplicated recursive processes to do the same work.
It would be a waste of the time.

Another trick is that we could use ```n = n / 2``` in both cases
($$n$$ is odd or even) since the result of ```n = (n - 1) / 2``` is same
as ```n = n / 2``` in _C/C++_'s world if ```n``` is an **odd** integer.

Thus, we can rewrite the code into:

```cpp
uint64_t fib(unsigned int n)
{
  if (n == 0) {
    return 0; // F(0) = 0.
  } else if (n <= 2) {
    return 1; // F(1) = F(2) = 1.
  }

  unsigned int k = n / 2; // k = n/2 if n is even. k = (n-1)/2 if n is odd.
  uint64_t a = fib(k);
  uint64_t b = fib(k + 1);

  if (n % 2) { // By F(n) = F(2k+1) = F(k+1)^2 + F(k)^2
    return a * a + b * b;
  } else { // By F(n) = F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    return a * (2 * b - a);
  }
}
```

#### Memoization

Do we save all duplicated task now? No.
Suppose we need to find $$F_6$$, then we need to get $$F_3, F_4$$ ....

$$
\begin{matrix}
 & & & & & 6 & & & & & & & \\
 & & & & \diagup & & \diagdown & & & & & & \\
 & & & \diagup & & & & \diagdown & & & & & \\
 & & 3 & & & & & & 4 & & & & \\
 & \diagup & & \diagdown & & & & \diagup & & \diagdown & & & \\
 1 & & & & 2 & & 2 & & & & 3 & & \\
 & & & & & & & & & \diagup & & \diagdown & \\
 & & & & & & & & 1 & & & & 2
\end{matrix}
$$

It's clear that we have a duplicated $$F_1, F_2, F_3$$ on above figure.
$$F_1, F_2$$ can return value directly, while $$F_3$$ can not.
Therefore, the sub-tree(sub-process) whose root is $$F_3$$ will be executed twice.

The larger the $$n$$ is, the more duplicated sub-process will be executed.
To avoid the waste, we can add an _array_ to save all the calculated value.
We check the _array_ first when $$F_n$$ is calculated.
If there is already a saved value in the _array_ at $$n$$,
then we can use it directly.
Otherwise, it will be calculated as usual.
It's called _memoization_.
We will save $$F_n$$ as the $$n$$ element in the _array_.

In this case, the $$F_n$$ is not calculated successively.
For example, to get $$F_6$$, we only need $$F_4, F_3, F_2, F_1, F_0$$.
We don't need $$F_5$$, so there is no value at the $$5$$ element in the _array_.
You can use _hash map_ instead of _array_ to avoid the waste of memory.
However, retrieving data from _array_ is faster than _hash map_,
so we apply _array_ in our sample code:

```cpp
const unsigned int SIZE = 100;
// 4 is not a fibonacci number, so using it as initialized value.
const uint64_t INIT = 4;
// In this case, F is not calculated successively. For example,
// To get F(6), we only need F(4), F(3), F(2), F(1), F(0) (no F(5)),
// so the other elements in F is still INIT.
// Another way is to use hash map(std::unordered_map), however,
// it will be slower.
uint64_t MEM[SIZE] = { [0 ... SIZE-1] = INIT };
uint64_t fib(unsigned int n)
{
  if (MEM[n] != INIT) {
    return MEM[n];
  }

  if (n == 0) {
    return (MEM[n] = 0); // F(0) = 0.
  } else if (n <= 2) {
    return (MEM[n] = 1); // F(1) = F(2) = 1.
  }

  unsigned int k = n / 2; // k = n/2 if n is even. k = (n-1)/2 if n is odd.
  uint64_t a = fib(k);
  uint64_t b = fib(k + 1);

  // By F(n) = F(2k+1) = F(k+1)^2 + F(k)^2, if n is odd.
  //    F(n) = F(2k) = F(k) * [ 2 * F(k+1) – F(k) ], if n is even.
  return (MEM[n] = (n % 2) ? a * a + b * b : a * (2 * b - a));
}
```

#### State vector
Although we can speed up the calculating by applying _memoization_ above,
the memory consumption with this approach grows with $$n$$.
Is it possible to use a fixed memory no matter how big $$n$$ is?
The answer is yes. Actually, we could just use a two-elements array to do it.

From the formula, we can calculate $$[F_{2n}, F_{2n+1}]$$ from $$[F_n, F_{n+1}]$$.
For example, to calculate $$F_{10}$$, we need $$F_5, F_6$$.
To calculate $$F_5$$, we need $$F_2, F_3$$.
To calculate $$F_2$$, we need $$F_1, F_0$$.
To calculate $$F_1$$, we need $$F_0, F_1$$
(so we need to stop here since $$F_1$$ is the dead end).

However, how do we get $$F_6$$
when we only have $$F_2, F_3$$ to calculate $$F_5$$?
Or how to get $$F_3$$
when we only have $$F_0, F_1$$ to calculate $$F_2$$ ...?

By applying $$n = 2$$ to formula, we can use $$F_2, F_3$$ to get $$F_4, F_5$$.
Then we can get $$F_6 = F_4 + F_5$$.

Thus, we are able to get $$F_{10}$$ by the following procedure:

$$
\require{AMScd}
\begin{CD}
\left(
  \begin{array}{c}
    F_0 \\
    F_1
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_1 \\
    F_2
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_2 \\
    F_3
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_5 \\
    F_6
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_{10} \\
    F_{11}
  \end{array}
\right)
\end{CD}
$$

Thus, we could keep using two-elements array
for $$\begin{bmatrix} F_n \\ F_{n+1} \end{bmatrix}$$
to compute what we want and update it step by step.

But how to determine the state we should update from $$[F_n, F_{n + 1}]$$,
$$[F_{2n}, F_{2n + 1}]$$ or $$[F_{2n + 1}, F_{2n + 2}]$$ ?

It's simple. If $$n$$ is even, we need to find $$F_k$$
, where $$k = \frac{n}{2}$$ since $$n = 2x$$.
Then we can use $$[F_k, F_{k+1}] = [F_{n/2}, F_{n/2 + 1}]$$
to calculate $$[F_{2k}, F_{2k + 1}] = [F_n, F_{n + 1}]$$.

Otherwise, if $$n$$ is odd, we need to find $$F_k$$
, where $$k = \frac{n-1}{2}$$ since $$n = 2k + 1$$.
Then we can use $$[F_k, F_{k+1}] = [F_{(n-1)/2}, F_{(n-1)/2 + 1}]$$
to calculate $$[F_{2k}, F_{2k+1}] = [F_{n-1}, F_n]$$
and then get $$[F_n, F_{n + 1}]$$ by $$[F_n, F_{n-1} + F_n]$$.

In summary, the procedure can be organized as fellows:

|                 $$k_i$$ |    $$k_1$$ | $$k_2$$ | $$k_3$$ | $$k_4$$ | $$k_5$$ |
| ----------------------- | ---------- | ------- | ------- | ------- | ------- |
| $$n(= k_i)$$            |         10 |       5 |       2 |       1 |       0 |
| $$n$$ is odd            |            |       v |         |       v |         |
| $$F_{2k_{i+1}}$$        |            | $$F_4$$ |         | $$F_0$$ |         |
| $$F_n$$                 | $$F_{10}$$ | $$F_5$$ | $$F_2$$ | $$F_1$$ | $$F_0$$ |
| $$F_{n+1}$$             |            | $$F_6$$ | $$F_3$$ | $$F_2$$ | $$F_1$$ |

The last two rows, $$F_n, F_{n+1}(= F_{k_i}, F_{k_i+1})$$, are the state vector
that contains our answer.

The first row $$n$$, is the index of the first element
of the state vector $$F_n$$.
The second row indicates that whether $$n$$ is odd or not.
If $$n$$ is odd(recall what we discuss above),

From the top-down perspective, to get $$F_{10}$$, we need $$F_5, F_6$$.
To get $$F_5, F_6$$, we need $$F_2, F_3$$.
To get $$F_2, F_3$$, we need $$F_1, F_2$$.
To get $$F_1, F_2$$, we need $$F_0, F_1$$.
We will demonstrate how we do it recursively below.

From the bottom-up perspective, we can use $$F_0, F_1$$
to get $$F_0, F_1, F_2$$,
then $$F_1, F_2$$ to get $$F_2, F_3$$,
$$F_2, F_3$$ to get $$F_4, F_5, F_6$$,
$$F_5, F_6$$ to get $$F_10$$.
We will demonstrate how we do it in iterative section.

The recursive approach is easier to understand.
By what we summarized above, the simplest implementation will be:

```cpp
// Set f[0], f[1] to F(n), F(n+1).
void fib_helper(unsigned int n, uint64_t f[]);

// 4 is not a fibonacci number, so using it as initialized value.
const uint64_t INIT = 4;

uint64_t fib(unsigned int n)
{
  uint64_t f[2] = { INIT, INIT };
  fib_helper(n, f);
  return f[0];
}

void fib_helper(unsigned int n, uint64_t f[])
{
  if (n == 0) {
    f[0] = 0; f[1] = 1;
    return;
  }

  unsigned int k = 0;
  if (n % 2) {
    k = (n - 1) / 2;
    fib_helper(k, f);
    uint64_t a = f[0];            // F(k) = F((n-1)/2)
    uint64_t b = f[1];            // F(k + 1) = F((n- )/2 + 1)
    uint64_t c = a * (2 * b - a); // F(n-1) = F(2k) = F(k) * [2 * F(k + 1) - F(k)]
    uint64_t d = a * a + b * b;   // F(n) = F(2k + 1) = F(k)^2 + F(k+1)^2
    f[0] = d;                     // F(n)
    f[1] = c + d;                 // F(n+1) = F(n-1) + F(n)
  } else {
    k = n / 2;
    fib_helper(k, f);
    uint64_t a = f[0];            // F(k) = F(n/2)
    uint64_t b = f[1];            // F(k + 1) = F(n/2 + 1)
    f[0] = a * (2 * b - a);       // F(n) = F(2k) = F(k) * [2 * F(k + 1) - F(k)]
    f[1] = a * a + b * b;         // F(n + 1) = F(2k + 1) = F(k)^2 + F(k+1)^2
  }
}
```

The above ```fib_helper``` is quite tedious,
we can be simplify it into:
```cpp
// Set f[0], f[1] to F(n), F(n+1).
void fib_helper(unsigned int n, uint64_t f[])
{
  if (!n) {
    f[0] = 0;
    f[1] = 1;
    return;
  }

  fib_helper(n / 2, f);
  // k = floor(n/2), so k = n / 2 if n is even, k = (n - 1) / 2 if n is odd.
  uint64_t a = f[0]; // F(k)
  uint64_t b = f[1]; // F(k+1)

  uint64_t c = a * (2 * b - a); // F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
  uint64_t d = a * a + b * b;   // F(2k+1) = F(k+1)^2 + F(k)^2

  if (n % 2) {    // k = (n - 1) / 2, so F(2k) = F(n-1), F(2k+1) = F(n).
    f[0] = d;     // F(n) = F(2k+1).
    f[1] = c + d; // F(n+1) = F(n-1) + F(n) = F(2k) + F(2k+1).
  } else {        // k = n / 2, so F(2k) = F(n), F(2k+1) = F(n+1).
    f[0] = c;     // F(n) = F(2k).
    f[1] = d;     // F(n+1) = F(2k).
  }
}
```

You could also replace _array_ with _std::vector_,
so the code will looks more elegant.
However, it will be slower than using _array_ directly.
```cpp
// Return vector [ F(n), F(n+1) ].
std::vector<uint64_t> fib_helper(unsigned int n);

uint64_t fib(unsigned int n)
{
  return fib_helper(n)[0];
}

std::vector<uint64_t> fib_helper(unsigned int n)
{
  if (!n) {
    // [F(0), F(1)] = [0 , 1]
    return { 0 , 1 };
  }

  std::vector<uint64_t> f(fib_helper(n / 2));
  // k = floor(n/2), so k = n / 2 if n is even, k = (n - 1) / 2 if n is odd.
  uint64_t a = f[0]; // F(k)
  uint64_t b = f[1]; // F(k+1)

  uint64_t c = a * (2 * b - a); // F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
  uint64_t d = a * a + b * b;   // F(2k+1) = F(k+1)^2 + F(k)^2

  if (n % 2) { // k = (n - 1) / 2, so F(2k) = F(n-1), F(2k+1) = F(n).
    // [F(n), F(n+1)] = [F(2k+1), F(2k+2)] = [F(2k+1), F(2k) + F(2k+1)]
    return { d, c + d };
  } else { // k = n / 2, so F(2k) = F(n), F(2k+1) = F(n+1).
    // [F(n), F(n+1)] = [F(2k), F(2k+1)].
    return { c, d };
  }
}
```

### Iterative

The recursive approach is implemented from the top-down perspective.
We could also do it in bottom-up way.

To convert the recursive steps into an iterative loop,
we need to find the _initialized state_ and the _stop condition_.
In the recursive approach, no matter what $$n$$ is, the final state vector
(when the recursive steps stops) is always $$[F_0, F_1]$$,
, and it must be called from calculating the state $$$$[F_1, F_2]$$$$.
Recall how we calculate $$F_{10}$$:

- We recursively calculate $$n = \lfloor \frac{n}{2} \rfloor$$ from $$n = 10$$,
  - then $$n = \lfloor \frac{10}{2} \rfloor = 5$$,
  - then $$n = \lfloor \frac{5}{2} \rfloor = 2$$,
  - then $$n = \lfloor \frac{2}{2} \rfloor = 1$$,
  - then stop recursive steps when $$n = \lfloor \frac{1}{2} \rfloor = 0$$.
- Next, we get the state vector $$[F_n, F_{n+1}]$$ for $$n = 0$$,
  - then return in the same route to calculate the state vector for $$n = 1$$,
  - then for $$n = 2$$
  - then for $$n = 5$$,
  - and finally get the answer for $$n = 10$$.

$$
\require{AMScd}
\begin{CD}
\left(
  \begin{array}{c}
    F_0 \\
    F_1
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_1 \\
    F_2
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_2 \\
    F_3
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_5 \\
    F_6
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_{10} \\
    F_{11}
  \end{array}
\right)
\end{CD}
$$

The recursive steps are used to get the route
from $$n = 0$$ to $$1, 2, 5, 10$$,
then calculate $$[F_n, F_{n+1}]$$ for each $$n$$.

To remove the recursive steps, we need to have a way to compute the route.
We can use a _stack_ to track the change for $$n$$, starting push $$n$$
from $$n = 10$$, then $$n = 5$$, $$n = 2$$, $$n = 1$$, $$n = 0$$,
then the route can be get from popping them from $$0$$ to $$10$$.

Thus, the _initialized state_ is $$n = 0$$
and the _stop condition_ is to check whether the stack is empty.

(Using _stack_ is one common approach to
convert recursive code into the iterative one.)

```cpp
uint64_t fib(unsigned int n)
{
  // To compute the route from n, n/2, ... 0.
  std::stack<unsigned int> s;
  while(n) {
    s.push(n);
    n /= 2; // n = floor(n/2)
  }
  s.push(n); // n = 0 now.

  uint64_t a; // F(n)
  uint64_t b; // F(n+1)
  while (!s.empty()) {
    unsigned int m = s.top(); s.pop();

    if (m == 0) { // Initializing a, b.
      a = 0; // F(0) = 0
      b = 1; // F(1) = 1
      continue;
    }

    // Let k = floor(m/2), so `a` is F(k) and `b` is F(k+1) now.
    // k = m/2, if m is even. k = (m-1)/2, if m is odd.
    uint64_t c = a * (2 * b - a); // F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    uint64_t d = a * a + b * b;   // F(2k+1) = F(k)^2 + F(k+1)^2

    if (m % 2) {  // m = 2k+1:
      a = d;      //  F(m) = F(2k+1)
      b = c + d;  //  F(m+1) = F(m) + F(m-1) = F(2k+1) + F(2k)
    } else {      // m = 2k:
      a = c;      //  F(m) = F(2k)
      b = d;      //  F(m+1) = F(2k+1)
    }
  }

  return a;
}
```

The above code is a bit ugly for simulating the recursive steps like:
$$
\require{AMScd}
\underbrace{
\begin{CD}
\left(
  \begin{array}{c}
    \boldsymbol{a} = F_0 \\
    \boldsymbol{b} = F_1
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_1 \\
    F_2
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_2 \\
    F_3
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_5 \\
    F_6
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_{10} \\
    F_{11}
  \end{array}
\right)
\end{CD}
}_{loop}
$$

The _initialized state_ is usually set outside of the loop directly like below:

```cpp
...
uint64_t a = 0; // F(0) = 0
uint64_t b = 1; // F(1) = 1
while (!s.empty()) {
  ...
}
...
```

$$
\require{AMScd}
\begin{CD}
\left(
  \begin{array}{c}
    \boldsymbol{a} = F_0 \\
    \boldsymbol{b} = F_1
  \end{array}
\right)
\end{CD}
\underbrace{
\begin{CD}
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_1 \\
    F_2
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_2 \\
    F_3
  \end{array}
\right)
@>{2n+1, 2n+2}>>
\left(
  \begin{array}{c}
    F_5 \\
    F_6
  \end{array}
\right)
@>{2n, 2n+1}>>
\left(
  \begin{array}{c}
    F_{10} \\
    F_{11}
  \end{array}
\right)
\end{CD}
}_{loop}
$$

Since _initialized state_ is set before the loop,
we should start the route from $$n = 1$$ to $$2, 5, 10$$:
```cpp
std::stack<unsigned int> s;
while (n) {
  s.push(n);
  n /= 2;
}
// No `s.push(n); // n = 0 now.` here!
```

Therefore, the code will be:
```cpp
uint64_t fib(unsigned int n)
{
  std::stack<unsigned int> s;
  while (n) {
    s.push(n);
    n /= 2;
  }

  uint64_t a = 0; // F(0) = 0
  uint64_t b = 1; // F(1) = 1
  while (!s.empty()) {
    unsigned int m = s.top(); s.pop();

    // Let k = floor(m/2), so `a` is F(k) and `b` is F(k+1) now.
    // k = m/2, if m is even. k = (m-1)/2, if m is odd.
    uint64_t c = a * (2 * b - a); // F(2k) = F(k) * [ 2 * F(k+1) – F(k) ]
    uint64_t d = a * a + b * b;   // F(2k+1) = F(k)^2 + F(k+1)^2

    if (m % 2) {  // m = 2k+1:
      a = d;      //  F(m) = F(2k+1)
      b = c + d;  //  F(m+1) = F(m) + F(m-1) = F(2k+1) + F(2k)
    } else {      // m = 2k:
      a = c;      //  F(m) = F(2k)
      b = d;      //  F(m+1) = F(2k+1)
    }
  }

  return a;
}
```
Another trick is that you can replace ```n /= 2``` by ```n >>= 1```
and ```m % 2``` by ```m & 1```

All the above code are on [gist here][gist].

[gist]: https://gist.github.com/ChunMinChang/f80ef4decca23b88df16f2f7846049b6 "Calculating Fibonacci Numbers by Fast Doubling"
