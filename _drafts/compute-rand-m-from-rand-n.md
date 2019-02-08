---
layout: post
title: Compute Rand M from Rand N
category: [Math]
tags: [Rust, Puzzles]
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
since the range of $$[0, M)$$ is wider than $$[0, N)$$ *(really? keep reading!)*.

To get a random number in $$[0, M)$$,
we need to **enlarge** the range of the numbers produced
by the given generator. How to do that ?

OK, the number of results from the the generator is $$N$$ now.
How to enlarge the range of results ?
Should we generate two numbers and do some math computation ?

How many results we have if we generate two numbers ?
If we see the results as permutations, it's $$N^2$$.

When $$N = 2$$, the results for generating two numbers can be:

| 1st num | 2nd num | probability                                     |
| ------- | ------- | ----------------------------------------------- |
| 0       | 0       | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 0       | 1       | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 1       | 0       | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |
| 1       | 1       | $$\frac{1}{2} \cdot \frac{1}{2} = \frac{1}{4}$$ |



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

## Compute Rand *M* from Rand *N*

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

Replace
```rs
r = r * x + rand(x);
```
bt
```rs
r = r << 1 | rand(2);
```

As a result the program is:
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

[rand_small_from_big]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=798f79143be66cecd08f81b48a97dd4b "Compute Rand([0, s)) from Rand([0, b)), where b, s are integers and b >= s"
[rand_big_from_small]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=fb12e41948db39ebf49340a0a151529e "Compute Rand([0, b)) from Rand([0, s)), where b, s are integers and b >= s"
[rand_m_from_n]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ad02e341de49665aeef118bdbb10aae9 "Compute Rand([0, m)) from Rand([0, n)), where m, n are integers"
[rand_from_2]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=5aa2d3ae1fe8e528628ca6fafbed5b86 "Compute Rand([0, k)) from Rand([0, 1]), where m, n are integers"



