---
layout: post
title: Exponentiation by squaring
category: [Algorithm]
tags: [Recursion, Dynamic Programming]
mathjax: true
comments: true
---
What is the time complexity of the computation for $$k^n$$?
We might intuitively think it's $$O(n)$$.
In fact, it can be done in $$O(\log n)$$
by [exponentiation by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring).

The key idea is:

$$
k^n =
\begin{cases}
(k^\frac{n}{2})^2,  & \text{if $n$ is even} \\
k \cdot (k^\frac{n-1}{2})^2, & \text{if $n$ is odd}
\end{cases}
$$

or

$$
k^n =
\begin{cases}
(k^2)^\frac{n}{2},  & \text{if $n$ is even} \\
k \cdot (k^2)^\frac{n-1}{2}, & \text{if $n$ is odd}
\end{cases}
$$

By dividing $$n$$ to $$\frac{n}{2}$$ again and again,
then stop when $$n = 0$$,
we could solve $$k^n$$ in $$O(\log n)$$.

## Top-down Approach


### Recursive
If we apply the first conversion, we could get the following code:

```cpp
uint64_t pow1(unsigned int k, unsigned int n)
{
  if (!n) {
    return 1;
  }

  if (n % 2) {
    return k * pow1(k, (n - 1) / 2) * pow1(k, (n - 1) / 2);
  } else {
    return pow1(k, n / 2) * pow1(k, n / 2);
  }
}
```

The above program uses two same recursions, ```... * pow1(...) * pow1(...)```,
as the returned value, so it will duplicate two same stacks.
On the other hand, if we apply the second conversion,
then there is no duplicated stack needed.
It could save almost half computation time of the ```pow1```
since it only uses half recursions than ```pow1```.

```cpp
uint64_t pow2(unsigned int k, unsigned int n)
{
  if (!n) {
    return 1;
  }

  if (n % 2) {
    return k * pow2(k * k, (n - 1) / 2);
  } else {
    return pow2(k * k, n / 2);
  }
}
```

The above code still could be simplified.
The result of the division $$y = \frac{x}{2}$$ is
actually $$y = \lfloor \frac{x}{2} \rfloor$$ in the world of *C* and *C++*,
if $$y$$ is an integer.

That is, if ```n``` is an **odd** integer,
then the result of ```n = (n - 1) / 2``` is same as ```n = n / 2```.

Thus, the code could be simplified as follows:

```cpp
uint64_t pow3(unsigned int k, unsigned int n)
{
  if (!n) {
    return 1;
  }

  uint64_t x = pow3(k * k, n / 2);
  // so x = (k^2)^(n/2),      if n is even
  //     or (k^2)^((n-1)/2),  if n is odd

  if (n % 2) {
    x = x * k; // so x = k * (k^2)^((n-1)/2)
  }
  return x;
}
```

Another trick is that We could replace ```a = b / 2``` by ```a = b >> 1```
and ```a = b % 2``` by ```a = b & 0x01```.
(But I guess your compiler might already do that for you.)

```cpp
uint64_t pow4(unsigned int k, unsigned int n)
{
  if (!n) {
    return 1;
  }

  uint64_t x = pow4(k * k, n >> 1);
  return (n & 1) ? x * k : x;
}
```

### Iterative
If we could rewrite a recursive algorithm into an iterative version,
it usually run faster.

If $$n$$ is always even, then it's easy to calculate in a loop.
For example, when $$k = 3, n = 8$$, we can calculate $$k^n = 3^8$$
by $$3^8 = (3^2)^4 = ((3^2)^2)^2 = (((3^2)^2)^2)^1$$.

Thus, we can find $$k^n$$ by:
1. $$k_0 = 3, n_0 = 8$$, now $$k^n = k_0^{n_0}$$
2. $$k_1 = k_0^2 = 3^2, n_1 = \frac{n_0}{2} = 4$$, now $$k^n = k_1^{n_1}$$
3. $$k_2 = (k_1)^2 = 3^4, n_2 = \frac{n_1}{2} = 2$$, now $$k^n = k_2^{n_2}$$
4. $$k_3 = (k_2)^2 = 3^8, n_3 = \frac{n_2}{2} = 1$$, now $$k^n = k_3^{n_3}$$
5. $$n_3 = 1$$, so $$k^n = k_3^{n_3} = k_3 = 3^8$$

On the other hand, if $$n$$ is not always even,
then we need to deal with the single leading $$k$$
in the $$k \cdot (k^2)^\frac{n-1}{2}$$,
which will not used to square.
For example, when $$k = 3, n = 7$$, we can calculate $$k^n = 3^7$$
by $$3 \cdot (3^2)^3 = 3 \cdot (3^2 \cdot ((3^2)^2)^1)$$.

In this case, we need one more variable $$r$$
to track the single leading $$k$$.
That is, we can find $$k^n$$ by:
1. $$k_0 = 3, n_0 = 7, r_0 = 1$$, now $$k^n = r_0 \cdot k_0^{n_0}$$
2. $$k_1 = k_0^2 = 3^2, n_1 = \frac{n_0 - 1}{2} = 3, r_1 = r_0 \cdot k_0 = 3$$,
now $$k^n = r_1 \cdot k_1^{n_1}$$
3. $$k_2 = k_1^2 = 3^4, n_2 = \frac{n_1 - 1}{2} = 1, r_2 = r_1 \cdot k_1 = 3^3$$,
now $$k^n = r_2 \cdot k_2^{n_2}$$
4. $$n_2 = 1$$, so $$k^n = r_2 \cdot k_2^{n_2} = r_2 \cdot k_2 = 3^3 \cdot 3^4$$

Wrapping up the above ideas, we could summarize the following code:
```cpp
uint64_t pow5(unsigned int k, unsigned int n)
{
  if (!n) {
    return 1;
  }

  uint64_t r = 1; // The remaining part for the squaring.
  while (n > 1) {
    if (n % 2) {
      r *= k;
      k *= k;
      n = (n - 1) / 2;
    } else {
      k *= k;
      n = n / 2;
    }
  }

  return r * k;
}
```

The $$r$$ is the product of all the single leading $$k$$.
The loop finishes when $$n = 1$$ in above code and return $$r \cdot k$$.

We could see there is a duplicated ```r * k``` in above.
If we keep looping when $$n = 1$$, then the ```r = r * k``` is our final answer.
Moreover, when $$n = 0$$, the initial ```r = 1``` is also correct,
so the beginning ```if``` could be saved.
However, we will waste a little time to compute the useless ```k = k * k```.

```cpp
uint64_t pow6(unsigned int k, unsigned int n)
{
  // The `r` should be the remaining part for the squaring(in pow5).
  // However, we notice that the `r * k` is duplicated in pow5. We will get
  // the answer by `r * k` when n = 1. If we keep looping when n = 1,
  // `r` is our answer. Nevertheless, we will waste time to do `k *= k`
  // when n = 1.
  uint64_t r = 1;

  while (n) {
    if (n % 2) {
      r *= k;
      k *= k;
      n = (n - 1) / 2;
    } else {
      k *= k;
      n = n / 2;
    }
  }

  return r;
}
```

There is different angle to see the above algorithm.
Actually, we can define $$k^n$$ by:

$$
k^n =
\begin{cases}
r \cdot (k^2)^\frac{n}{2}, r = 1 & \text{if $n$ is even} \\
r \cdot (k^2)^\frac{n-1}{2}, r = k & \text{if $n$ is odd}
\end{cases}
$$

Thus, we could also use $$r$$ to find $$k^n$$.
By the example above when $$k = 3, n = 8$$:
1. $$k_0 = 3, n_0 = 8, r_0 = 1$$, now $$k^n = r_0 \cdot k_0^{n_0}$$
2. $$k_1 = k_0^2 = 3^2, n_1 = \frac{n_0}{2} = 4, r_1 = r_0 \cdot 1 = 1$$, now $$k^n = r_1 \cdot k_1^{n_1}$$
3. $$k_2 = (k_1)^2 = 3^4, n_2 = \frac{n_1}{2} = 2, r_2 = r_1 \cdot 1 = 1$$, now $$k^n = r_2 \cdot k_2^{n_2}$$
4. $$k_3 = (k_2)^2 = 3^8, n_3 = \frac{n_2}{2} = 1, r_3 = r_2 \cdot 1 = 1$$, now $$k^n = r_3 \cdot k_3^{n_3}$$
5. $$k_4 = (k_3)^2 = 3^16, n_4 = \frac{n_3 - 1}{2} = 0, r_4 = r_3 \cdot k_3 = 3^8$$,
now $$k^n = r_4 \cdot k_4^{n_4}$$
6. $$n_4 = 0$$, so $$k^n = r_4 = 3^8$$

We could see a more general case when $$k = 3, n = 10$$:
1. $$k_0 = 3, n_0 = 10, r_0 = 1$$, now $$k^n = r_0 \cdot k_0^{n_0}$$
2. $$k_1 = k_0^2 = 3^2, n_1 = \frac{n_0}{2} = 5, r_1 = r_0 \cdot 1 = 1$$, now $$k^n = r_1 \cdot k_1^{n_1}$$
3. $$k_2 = (k_1)^2 = 3^4, n_2 = \frac{n_1 - 1}{2} = 2, r_2 = r_1 \cdot k_1 = 3^2$$, now $$k^n = r_2 \cdot k_2^{n_2}$$
4. $$k_3 = (k_2)^2 = 3^8, n_3 = \frac{n_2}{2} = 1, r_3 = r_2 \cdot 1 = 3^2$$, now $$k^n =  r_3 \cdot k_3^{n_3}$$
5. $$k_4 = (k_3)^2 = 3^{16}, n_4 = \frac{n_3 - 1}{2} = 0, r_4 = r_3 \cdot k_3 = 3^{10}$$,
now $$k^n = r_4 \cdot k_4^{n_4}$$
6. $$n_4 = 0$$, so $$k^n = r_4 = 3^{10}$$

In summary, the whole process can be organized into following table:

| round $$i$$ |      0 |       1 |       2 |       3 |          4 |
| ----------- | ------ | ------- | ------- | ------- | ---------- |
|       $$n$$ | $$10$$ |   $$5$$ |   $$2$$ |   $$1$$ |      $$0$$ |
|       $$k$$ |  $$3$$ | $$3^2$$ | $$3^4$$ | $$3^8$$ | $$3^{16}$$ |
|       $$r$$ |  $$1$$ |   $$1$$ | $$3^2$$ | $$3^2$$ | $$3^{10}$$ |

By calculating $$n_{i+1} = \lfloor \frac{n_i}{2} \rfloor$$,
$$k_{i+1} = {k_i}^2$$ and $$r_{i+1} = r_i \cdot c_i$$,
where $$c_i = k_i$$ when $$n_i$$ is odd or $$c_i = 1$$ when $$n_i$$ is even,
we can get the answer by $$k^n = r_j$$ when $$n_j = 0$$ for some $$j$$.

As a result, the algorithm is:

```cpp
uint64_t pow6(unsigned int k, unsigned int n)
{
  uint64_t r = 1;

  while (n) {
    if (n % 2) {
      r *= k;
      k *= k;
      n = (n - 1) / 2;
    } else {
      r *= 1; // This could be saved!
      k *= k;
      n = n / 2;
    }
  }

  return r;
}
```

Like what we mentioned in recursive part,
If ```n``` is an **odd** integer,
then the result of ```n = (n - 1) / 2``` is same as ```n = n / 2```.
nd we can also replace ```a = b / 2``` by ```a = b >> 1```
and ```a = b % 2``` by ```a = b & 0x01```.
Finally, the algorithm can be shorten as:

```cpp
uint64_t pow7(unsigned int k, unsigned int n)
{
  uint64_t r = 1;
  while (n) {
    if (n & 1) { // n % 2
      r *= k;
    }
    k *= k;
    n >>= 1; // n = n / 2;
  }

  return r;
}
```

## Bottom-up Approach

The top-down approach is to calculate the value
from $$x = n$$ to $$x = 0$$ repeatedly,
where $$x \leftarrow \lfloor \frac{x}{2} \rfloor$$,
and the $$r$$'s value is updated when $$x$$ is odd.

The $$j$$ is changed like:

$$
\require{AMScd}
\begin{CD}
10
@>{n = \frac{n}{2}}>>
5
@>{n = \frac{n-1}{2}}>>
2
@>{n = \frac{n}{2}}>>
1
@>{n = \frac{n-1}{2}}>>
0
\end{CD}
$$

To convert it into bottom-up approach,
we need to run in the __opposite__ direction:

$$
\require{AMScd}
\begin{CD}
0
@>{n = 2n + 1}>>
1
@>{n = 2n}>>
2
@>{n = 2n + 1}>>
5
@>{n = 2n}>>
10
\end{CD}
$$

Suppose we have $$k^{n_j}$$,
where $$n_j = \lfloor \frac{n_{j+1}}{2} \rfloor$$,
then we can calculate $$k^{n_{j+1}}$$ by:

$$
k^{n_{j+1}} =
\begin{cases}
k^{2 n_j} = (k^{n_j})^2,  & \text{if $n_{j+1}$ is even} \\
k^{2 n_j + 1} = k \cdot (k^{n_j})^2, & \text{if $n_{j+1}$ is odd}
\end{cases}
$$

since

$$
n_{j+1} =
\begin{cases}
2 n_j,  & \text{if $n_{j+1}$ is even} \\
2 n_j + 1, & \text{if $n_{j+1}$ is odd}
\end{cases}
$$

Thus, if we have the track the changing of $$n_j$$,
then we can use a single variable $$a$$ to calculate $$k^{n_j} = a_j$$ by

$$
k^{n_{j+1}} = a_{j+1} =
\begin{cases}
(k^{n_j})^2 = {a_j}^2,  & \text{if $n_{j+1}$ is even} \\
k \cdot (k^{n_j})^2 = k \cdot {a_j}^2, & \text{if $n_{j+1}$ is odd}
\end{cases}
$$

, where $$n_j = 0, a_0 = 1$$.

| round $$j$$ |     0 |              1 |               2 |                  3 |                  4 |
| ----------- | ----- | -------------- | --------------- | ------------------ | ------------------ |
|       $$n$$ | $$0$$ |          $$1$$ |           $$2$$ |              $$5$$ |             $$10$$ |
|         odd |       |              v |                 |                  v |                    |
|       $$a$$ | $$1$$ | $$k\cdot{a_0}^2=k$$ | $${a_1}^2=k^2$$ | $$k\cdot{a_2}^2 = k^5$$ | $${a_3}^2=k^{10}$$ |


The only question now is how we could get $$n_j$$.
The changing of $$n_j$$ here is __opposite__ to the changing
of the recursive approach.
Thus, if we could push all the changing in recursive approach
into a _stack_, then we can pop them to get the opposite changing.
That is,

```cpp
// To track the variation of n.
std::stack<unsigned int> s;

// Get the n's changing in the recursive approach.
while(n) {
  s.push(n);
  n = n / 2;
}
// We lost 0 here, so we need to set the initial state for n_j = 0

// Initializing variable for n_j = 0 ...

// Get the opposite track in the recursive approach.
while (!s.empty()) {
  unsigned int x = s.top(); s.pop(); // Get the current n_j.
  // Calculate our answer here ...
}
```

Obviously, we have $$h = \lceil log_2 n \rceil$$ items in the _stack_,
so the loop will run $$h$$ rounds (the above $$j$$ is from $$0$$ to $$h$$).

In the case for $$n = 10$$, the bottom-up approach will run as follows:

$$
\require{AMScd}
\begin{CD}
0
\end{CD}
\underbrace{
\begin{CD}
@>{n = 2n + 1}>>
1
@>{n = 2n}>>
2
@>{n = 2n + 1}>>
5
@>{n = 2n}>>
10
\end{CD}
}_{loop}
$$

To calculate our answer,
we need to add a variable $$a$$
to keep tracking the $$k^{n_j} = a_j$$:

```cpp
uint64_t pow8(unsigned int k, unsigned int n)
{
  std::stack<unsigned int> s;
  while(n) {
    s.push(n);
    n >>= 1/*n /= 2*/;
  }

  uint64_t a = 1; // a = k^0 = 1
  while (!s.empty()) {
    unsigned int x = s.top(); s.pop();
    // Let y = floor(x/2), y = x/2 if x is even, y = (x-1)/2 if x is odd.
    // then a = k^y now.
    a *= a; // a = (k^y)^2 = k^(2y)
                  // x is even:
                  //   a = k^x = k^(2y)
    if (x & 1 /*x % 2*/) {  // x is odd:
      a *= k;     //   a = k^x = k^(2y+1) = k * k^(2y)
    }
  }

  return a;
}
```

To get even or odd the ```x``` is, the ```x``` is checked iteratively,
where ```x = n >> j``` and ```j``` is the times we have looped,
by ```x & 1``` in the ```while (!s.empty())```.
In other word, we are actually checking
from the __lowest bit to highest bit__ of ```x```.

Thus, the code could be rewritten into:

```cpp
uint64_t pow9(unsigned int k, unsigned int n)
{
  // The position of the highest bit of n.
  // So we need to loop `h` times to get the answer.
  // Example: n = (Dec)50 = (Bin)00110010, then h = 6.
  //                               ^ 6th bit from right side
  unsigned int h = 0;
  for (unsigned int i = n ; i ; ++h, i >>= 1);

  uint64_t a = 1; // a = k^0 = 1
  // There is only one `1` in the bits of `mask`. The `1`'s position is same as
  // the highest bit of n(mask = 2^(h-1) at first), and it will be shifted right
  // iteratively to do `AND` operation with `n` to check `n_j` is odd or even,
  // where n_j is defined below.
  for (unsigned int mask = 1 << (h - 1) ; mask ; mask >>= 1) { // Run h times!
    // Let j = h-i (looping from i = 1 to i = h), n_j = floor(n / 2^j) = n >> j
    // (n_j = n when j = 0), x = floor(n_j / 2), then a = k^x now.
    a *= a; // a = (k^x)^2 = k^(2x)
    // n_j is even: x = n_j / 2 => n_j = 2x
    //   a = k^(n_j) = k^(2x)
    if (n & mask) { // n_j is odd: x = (n_j - 1) / 2 => n_j = 2x + 1
      a *= k;       //   a = k^(n_j) = k^(2x+1) = k * k^(2x)
    }
  }

  return a;
}
```

All the above code are on [gist here][gist].

[gist]: https://gist.github.com/ChunMinChang/9753c72e2441343e14757f5a9ac95a98 "Exponentiation by squaring"
