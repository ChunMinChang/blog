---
layout: post
title: Compute Rand M from Rand N
category: [Puzzle, Math, Algorithm]
tags: [Random, Probabilitiy]
mathjax: true
comments: true
---

How to get a random number in *[0, M)* by a random number generator
that randomly throws a number in *[0, N)* with equal probability.

<!--read more-->

# $$Rand(M)$$ by $$Rand(N)$$

In this post, $$Rand \; K$$ or $$Rand(K)$$ denotes a random number generator
that produce a integer randomly in $$[0, K)$$
(or $$[0, K - 1]$$, from $$0$$ to $$K-1$$ inclusively)
with equal probability.

We can simply divide this problem into two cases: $$M \leq N$$ or $$M \gt N$$.

## $$M \leq N$$

If $$M \leq N$$, then the range $$[0, M)$$ is in the range $$[0, N)$$.
Therefore, the probability of a number choosed by $$Rand(N)$$ in $$[0, M)$$
is same. All of them are $$1/N$$.


| number     | probability     |
| ---------- | --------------- |
| $$0$$      | $$\frac{1}{N}$$ |
| $$1$$      | $$\frac{1}{N}$$ |
| $$2$$      | $$\frac{1}{N}$$ |
| $$\vdots$$ | $$\vdots$$      |
| $$M-1$$    | $$\frac{1}{N}$$ |
| $$\vdots$$ | $$\vdots$$      |
| $$N-1$$    | $$\frac{1}{N}$$ |


What we want is to develop a method
that can randomly choose a number in $$[0, M)$$ with equal probability.
We can leverage this fact.

The simplest way is to **re-produce** a number
if the number produced is out of the range we want ($$[0, M)$$).
That is, if the number produced is in $$[M, N)$$,
we **re-generate** a number.

Does it work? That's check the probability for each number.
For each round, the probability we need to re-generate a number is $$\frac{N - M}{N}$$.
Hence, the probability to get a number $$i$$ in the $$K$$ round
can be organized as the following table,
where $$i \in [0, M)$$ and $$K \in \mathbb{N}$$.


| round      | probability                                     |
| ---------- | ----------------------------------------------- |
| $$1$$      | $$\frac{1}{N}$$                                 |
| $$2$$      | $$\frac{N - M}{N} \cdot \frac{1}{N}$$           |
| $$3$$      | $${(\frac{N - M}{N})}^{2} \cdot \frac{1}{N}$$   |
| $$\vdots$$ | $$\vdots$$                                      |
| $$K$$      | $${(\frac{N - M}{N})}^{K-1} \cdot \frac{1}{N}$$ |


As a result, the probability to get the number $$i$$ is


$$
\sum_{k=1}^\infty {(\frac{N - M}{N})}^{k-1} \cdot \frac{1}{N}
= \frac{1}{N}
+ \frac{N - M}{N} \cdot \frac{1}{N}
+ {(\frac{N - M}{N})}^{2} \cdot \frac{1}{N}
+ \ldots
+ {(\frac{N - M}{N})}^{K-1} \cdot \frac{1}{N}
$$


This is same for all number $$i$$, where $$i \in [0, M)$$.


| number     | probability                                                       |
| ---------- | ----------------------------------------------------------------- |
| $$0$$      | $$\sum_{k=1}^\infty {(\frac{N - M}{N})}^{k-1} \cdot \frac{1}{N}$$ |
| $$1$$      | $$\sum_{k=1}^\infty {(\frac{N - M}{N})}^{k-1} \cdot \frac{1}{N}$$ |
| $$2$$      | $$\sum_{k=1}^\infty {(\frac{N - M}{N})}^{k-1} \cdot \frac{1}{N}$$ |
| $$\vdots$$ | $$\vdots$$                                                        |
| $$M-1$$    | $$\sum_{k=1}^\infty {(\frac{N - M}{N})}^{k-1} \cdot \frac{1}{N}$$ |


By now, we alreay know how to randomly generate a number in a smaller range
from a random number generator in a bigger range.

In brief, the algorithm is
1. Get a random number $$x$$ in $$[0, N)$$
2. If $$x$$ is in $$[0, M)$$, then return $$x$$
3. Otherwise, repeat from 1


The following *Rust* program is the method we developed above:

```rs
// Get a random number in [0, s) by a random number generator in [0, b),
// where both s, b are integers and s > 1, b > 1, s <= b
fn rand_small_from_big(s: u32, b: u32) -> u32 {
    assert!(s > 1 && b > 1 && s <= b);
    loop {
        // Get a random number in [0, b)
        let x = rand(b);
        // Return the random number if it is in [0, s)
        if x < s {
            return x;
        }
        // Repeating if the random number is in [s, b)
    }
}
```

See the live demo [here][rand_small_from_big].

## $$M \gt N$$

However, if $$M \gt N$$, our method above doesn't work,
since the range of $$[0, M)$$ is bigger than $$[0, N)$$ *(really? keep reading!)*.

To get a random number in $$[0, M)$$,
we need to **enlarge** the range of the numbers produced
by the given generator. How to do that ?

OK, the number of results from the the generator is $$N$$ now.
How to enlarge the range of results ?
Should we generate two numbers and do some magic math ?

How many results we have if we generate two numbers ?
If we see the results as **permutations**, it's $$N^2$$.

When $$N = 2$$, the results are $$(0, 0), (0, 1), (1, 0), (1, 1)$$.
The probability of each result is $$1/4$$.

| 1st | 2nd | probability                                     |
| --- | --- | ----------------------------------------------- |
| 0   | 0   | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 0   | 1   | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 1   | 0   | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 1   | 1   | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |

If the results can be mapped from $$(0, 0), (0, 1), (1, 0), (1, 1)$$
to $$0, 1, 2, 3$$, it means we **have a way** to enlarge the range of results
from $$[0, 1]$$ to $$[0, 1, 2, 3]$$.

Well, is it possible to do that ?
If we observe carefully,
it's natural to map $$(0, 0), (0, 1), (1, 0), (1, 1)$$ to $$0, 1, 2, 3$$.
If we see $$00, 01, 10, 11$$ as binary numbers $${00}_{2}, {01}_{2}, {10}_{2}, {11}_{2}$$,
then they are naturally $$0, 1, 2, 3$$.

The same approach also works
when we produce two numbers from random number generator whose range is $$[0, 3)$$.
There are $$3 \cdot 3 = 9$$ results: $$(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)$$.

| 1st | 2nd | probability                                     |
| --- | --- | ----------------------------------------------- |
| 0   | 0   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 0   | 1   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 0   | 2   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 1   | 0   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 1   | 1   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 1   | 2   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 2   | 0   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 2   | 1   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |
| 2   | 2   | $$\frac{1}{3} \cdot \frac{1}{3} = \frac{1}{9}$$ |

If we see them as the **base-$$3$$** numbers, they are naturally $$0, 1, 2, 3, 4, 5, 6, 7, 8$$.

By applying this generating-two-numbers approach for a random number generator with range in $$[0, N)$$,
the sequential results $$(i, j)$$, where $$i, j \in [0, N)$$,
can be mapped to $$0, 1, 2, \ldots, N^2 - 1$$ (range in $$[0, N^2)$$)
by treating them as **base-$$N$$** numbers $${ij}_{N}$$.

In general, for the random number generator with range in $$[0, N)$$,
there are $$N^k$$ sequential results for producing $$k$$ numbers.
The sequential results are $$(x_0, x_1, \ldots, x_{k-1})$$, where $$x_i \in [0, N)$$ and $$i \in [0, k)$$.
By taking the sequential results in **base-$$N$$** : $${(x_0 x_1 \ldots x_{k-1})}_{N}$$,
their valus are naturally $$0, 1, 2, \ldots, N^k$$.

| 1st        | 2nd        | $$\ldots$$ | kth        | probability       |
| ---------- | ---------- | ---------- | ---------- | ----------------- |
| 0          | 0          | $$\ldots$$ | 0          | $$\frac{1}{N^k}$$ |
| 0          | 0          | $$\ldots$$ | 1          | $$\frac{1}{N^k}$$ |
| 0          | 0          | $$\ldots$$ | 2          | $$\frac{1}{N^k}$$ |
| $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$        |
| 0          | 0          | $$\ldots$$ | N-1        | $$\frac{1}{N^k}$$ |
| $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$        |
| N-1        | N-1        | $$\ldots$$ | 0          | $$\frac{1}{N^k}$$ |
| N-1        | N-1        | $$\ldots$$ | 1          | $$\frac{1}{N^k}$$ |
| N-1        | N-1        | $$\ldots$$ | 2          | $$\frac{1}{N^k}$$ |
| $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$ | $$\vdots$$        |
| N-1        | N-1        | $$\ldots$$ | N-1        | $$\frac{1}{N^k}$$ |


When producing $$k$$ numbers by the random number generator with range in $$[0, N)$$,
the sequential results $$x_0, x_1, \ldots, x_{k-1}$$ can be mapped to a number in $$[0, N^k)$$
by encoding them as a **base-$$N$$** number $${(x_0 x_1 \ldots x_{k-1})}_{N}$$.

The idea can be implemented as the following program:
```rs
// Map the result of (x_0, x_1, ..., x_{k-1})
// to (x_0 x_1 ... x_{k-1}) in base-N number
let mut x = 0;
for _ in 0..k {
    // Repeat k times
    x = x * N + rand(N);
}
```

There are $$N^k$$ kinds of sequential results for producing $$k$$ numbers.
The probability for each one is $$\frac{1}{N^k}$$.
By this approach, we have a way to get one result
in the $$N^k$$ kinds of sequential results.
That is, we have a way to generate a random number in $$[0, N^k)$$
by the random number generator in $$[0, N)$$.

The original problem is to find a way to generate a random number in $$[0, M)$$
by the random number generator in $$[0, N)$$, where $$M \gt N$$.
Since we alreay know how to get a random number in a smaller range by
a random number generator in a bigger range
(the method developped in the $$M \leq N$$ case),
if we can find a $$k$$ such that $$N^k \geq M$$, then the problem can be solved!

The $$k$$ can be calculated by *logarithm*:

$$
N^k \geq M
\Rightarrow k \geq \log_N M
$$

The corresponding program is:
```rs
// Return minimal k such that N^k >= M
fn min_pow(N: u32, M: u32) -> u32 {
    let M_f64 = f64::from(M);
    let N_f64 = f64::from(N);
    M_f64.log(N_f64).ceil()) as u32
}
```

To sum up, if $$M \gt N$$, then
1. Find a $$k$$ such that $$N^k \geq M$$
2. Get the random number generator in $$[0, N^k)$$
3. Generate a random number in $$[0, M)$$ by the above generator
    1. Get a random number $$x$$ in $$[0, N^k)$$
    2. If $$x$$ is in $$[0, M)$$, then return $$x$$
    3. Otherwise, repeat from 3-1

```rs
// Get a random number in [0, b) by a random number generator in [0, s),
// where both s, b are integers and s > 1, b > 1, s <= b
fn rand_big_from_small(b: u32, s: u32) -> u32 {
    assert!(s > 1 && b > 1 && s <= b);
    // Get a minimal number p such that s^p >= b
    let p = min_pow(s, b);
    assert!(s.pow(p) >= b);
    loop {
        // Get a random number in [0, s^p)
        let r = rand_pow(s, p);
        // Return the random number if it is in [0, b)
        if r < b {
            return r;
        }
        // Repeating if the random number is in [b, s^p)
    }
}

// Get a random from [0, x^y)
fn rand_pow(x: u32, y: u32) -> u32 {
    assert!(x > 1 && y > 0);
    let mut r = 0;
    for _ in 0..y {
        // Repeat y times
        r = r * x + rand(x);
    }
    r
}

// Return minimal k such that x^k >= y
fn min_pow(x: u32, y: u32) -> u32 {
    (f64::from(y).log(f64::from(x)).ceil()) as u32
}
```

See the live demo [here][rand_big_from_small].

<!-- ### Enlarge from [0, 1, ..., N-1] to [0, 1, ..., N^k-1]
Look from another view ....

```rs
// Map the result of (x_0, x_1, ..., x_{k-1})
// to (x_0 x_1 ... x_{k-1}) in base-N number
let mut x = 0;
for _ in 0..k {
    // Repeat k times
    x = x * N + rand(N);
}
```

is actually enlarge a list from [0, 1, ..., N-1] to [0, 1, ..., N^k-1] -->


## Compute Rand *M* from Rand *N*

In fact, the method developped for $$M \leq N$$ case
is a special case in the method for $$M \gt N$$ case.

In the method for $$M \gt N$$ case,
the first step is to find a $$k$$ such that $$N^k \geq M$$.
If $$M \leq N$$, then $$k$$ is $$1$$.

Thus, the algorithm can be summarized as:
```rs
// Get a random number in [0, m) by a random number generator in [0, n)
fn rand_m_from_n(m: u32, n: u32) -> u32 {
    assert!(m > 1 && n > 1);
    // Get a minimal number p such that n^p >= m
    let p = min_pow(n, m);
    loop {
        // Get a random number in [0, n^p)
        let r = rand_pow(n, p);
        // Return the random number if it is in [0, m)
        if r < m {
            return r;
        }
        // Repeating if the random number is in [m, n^p)
    }
}

// Get a random from [0, x^y)
fn rand_pow(x: u32, y: u32) -> u32 {
    assert!(x > 1 && y > 0);
    let mut r = 0;
    for _ in 0..y {
        // Repeat y times
        r = r * x + rand(x);
    }
    r
}

// Return minimal k such that x^k >= y
fn min_pow(x: u32, y: u32) -> u32 {
    (f64::from(y).log(f64::from(x)).ceil()) as u32
}
```

See the live demo [here][rand_m_from_n].

### If *N* is 2

One mentionable trick is to replace
```rs
x = x * 2 + rand(2);
```
by
```rs
x = x << 1 | rand(2);
```
when $$N = 2$$.

As a result, the program is:
```rs
// Get a random number in [0, x) by a random number generator in [0, 1]
fn rand_from_2(x: u32) -> u32 {
    assert!(x > 1);
    // Get a minimal number p such that 2^p >= x
    let p = min_pow(x);
    loop {
        // Get a random number in [0, 2^p)
        let r = rand_pow(p);
        // Return the random number if it is in [0, x)
        if r < x {
            return r;
        }
        // Repeating if the random number is in [x, 2^p)
    }
}

// Get a random from [0, 2^k)
fn rand_pow(k: u32) -> u32 {
    assert!(k > 0);
    let mut r = 0;
    for _ in 0..k {
        // Repeat k times
        r = r << 1 | rand(2);
    }
    r
}

// Return minimal k such that 2^k >= x
fn min_pow(x: u32) -> u32 {
    (f64::from(x).log(f64::from(2)).ceil()) as u32
}
```

See the live demo [here][rand_from_2].

### How to check if the distribution is uniform
One easy way is to apply [*chi square test*][chi_sq_test].
The discussion can be found [here][test_uni_dist].
The [Testing a Random Number Generator][johndcook] chapter
in *John D. Cook*'s *Beautiful Testing* is also a great reference to read.

[rand_small_from_big]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=798f79143be66cecd08f81b48a97dd4b "Compute Rand([0, s)) from Rand([0, b)), where b, s are integers and b >= s"
[rand_big_from_small]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=fb12e41948db39ebf49340a0a151529e "Compute Rand([0, b)) from Rand([0, s)), where b, s are integers and b >= s"
[rand_m_from_n]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ad02e341de49665aeef118bdbb10aae9 "Compute Rand([0, m)) from Rand([0, n)), where m, n are integers"
[rand_from_2]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=5aa2d3ae1fe8e528628ca6fafbed5b86 "Compute Rand([0, k)) from Rand([0, 1]), where m, n are integers"

[chi_sq_test]: https://en.wikipedia.org/wiki/Chi-squared_test
[test_uni_dist]: https://math.stackexchange.com/questions/2435/is-there-a-simple-test-for-uniform-distributions
[johndcook]: https://www.johndcook.com/Beautiful_Testing_ch10.pdf


