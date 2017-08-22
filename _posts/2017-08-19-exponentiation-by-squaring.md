---
layout: post
title: Exponentiation by squaring
category: [Algorithm, Math]
tags: [power]
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

## Recursive approach

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

Thus, the code could be simplified as fellow:

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

## Iterative approach
If we could rewrite a recursive algorithm into an iterative version,
it usually run faster.

If $$n$$ is always even, then it's easy to calculate in a loop.
For example, when $$k = 3, n = 8$$, we can calculate $$k^n = 3^8$$
by $$3^8 = (3^2)^4 = ((3^2)^2)^2 = (((3^2)^2)^2)^1$$.

Thus, we can find $$k^n$$ by:
1. $$k_1 = 3, n_1 = 8$$, now $$k^n = k_1^{n_1}$$
2. $$k_2 = k_1^2 = 3^2, n_2 = \frac{n_1}{2} = 4$$, now $$k^n = k_2^{n_2}$$
3. $$k_3 = (k_2)^2 = 3^4, n_3 = \frac{n_2}{2} = 2$$, now $$k^n = k_3^{n_3}$$
4. $$k_4 = (k_3)^2 = 3^8, n_4 = \frac{n_3}{2} = 1$$, now $$k^n = k_4^{n_4}$$
5. $$n_4 = 1$$, so $$k^n = k_4^{n_4} = k_4 = 3^8$$


<br />
On the other hand, if $$n$$ is not always even,
then we need to deal with the single leading $$k$$
in the $$k \cdot (k^2)^\frac{n-1}{2}$$,
which will not used to square.
For example, when $$k = 3, n = 7$$, we can calculate $$k^n = 3^7$$
by $$3 \cdot (3^2)^3 = 3 \cdot (3^2 \cdot ((3^2)^2)^1)$$.

In this case, we need one more variable $$r$$
to track the single leading $$k$$.
That is, we can find $$k^n$$ by:
1. $$k_1 = 3, n_1 = 7, r_1 = 1$$, now $$k^n = r_1 \cdot k_1^{n_1}$$
2. $$k_2 = k_1^2 = 3^2, n_2 = \frac{n_1 - 1}{2} = 3, r_2 = r_1 \cdot k_1 = 3$$,
now $$k^n = r_2 \cdot k_2^{n_2}$$
3. $$k_3 = k_2^2 = 3^4, n_3 = \frac{n_2 - 1}{2} = 1, r_3 = r_2 \cdot k_2 = 3^3$$,
now $$k^n = r_3 \cdot k_3^{n_3}$$
4. $$n_3 = 1$$, so $$k^n = r_3 \cdot k_3^{n_3} = 3^3 \cdot 3^4$$

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
1. $$k_1 = 3, n_1 = 8, r_1 = 1$$, now $$k^n = r_1 \cdot k_1^{n_1}$$
2. $$k_2 = k_1^2 = 3^2, n_2 = \frac{n_1}{2} = 4, r_2 = r_1 \cdot 1 = 1$$, now $$k^n = r_2 \cdot k_2^{n_2}$$
3. $$k_3 = (k_2)^2 = 3^4, n_3 = \frac{n_2}{2} = 2, r_3 = r_2 \cdot 1$$, now $$k^n = r_3 \cdot k_3^{n_3}$$
4. $$k_4 = (k_3)^2 = 3^8, n_4 = \frac{n_3}{2} = 1, r_4 = r_3 \cdot 1$$, now $$k^n = r_4 \cdot k_4^{n_4}$$
5. $$k_5 = (k_4)^2 = 3^16, n_5 = \frac{n_4 - 1}{2} = 0, r_5 = r_4 \cdot k_4 = 3^8$$,
now $$k^n = r_5 \cdot k_5^{n_5}$$
6. $$n_5 = 0$$, so $$k^n = r_5 = 3^8$$

We could see a more general case when $$k = 3, n = 10$$:
1. $$k_1 = 3, n_1 = 10, r_1 = 1$$, now $$k^n = r_1 \cdot k_1^{n_1}$$
2. $$k_2 = k_1^2 = 3^2, n_2 = \frac{n_1}{2} = 5, r_2 = r_1 \cdot 1 = 1$$, now $$k^n = r_2 \cdot k_2^{n_2}$$
3. $$k_3 = (k_2)^2 = 3^4, n_3 = \frac{n_2 - 1}{2} = 2, r_3 = r_2 \cdot k_2 = 3^2$$, now $$k^n = r_3 \cdot k_3^{n_3}$$
4. $$k_4 = (k_3)^2 = 3^8, n_4 = \frac{n_3}{2} = 1, r_4 = r_3 \cdot 1 = 3^2$$, now $$k^n =  r_4 \cdot k_4^{n_4}$$
5. $$k_5 = (k_4)^2 = 3^{16}, n_5 = \frac{n_4 - 1}{2} = 0, r_5 = r_4 \cdot k_4 = 3^{10}$$,
now $$k^n = r_5 \cdot k_5^{n_5}$$
6. $$n_5 = 0$$, so $$k^n = r_5 = 3^{10}$$

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

All the above code are on [gist here][gist].

[gist]: https://gist.github.com/ChunMinChang/9753c72e2441343e14757f5a9ac95a98 "Exponentiation by squaring"
