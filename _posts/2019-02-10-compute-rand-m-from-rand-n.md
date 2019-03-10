---
layout: post
title: Compute Rand M from Rand N
category: [Puzzle, Math, Algorithm]
tags: [Random, Probabilitiy]
mathjax: true
comments: true
---

How to get a random number in *[0, M)* by a random number generator
that randomly throws a number in *[0, N)* with equal probability ?

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
is same. All of them are $$\frac{1}{N}$$.


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

Actually, we could make some improvement for this approach. 
By the above approach, it's very likely to re-produce the random number 
again and again when $$N$$ is much bigger than $$M$$.
The problem is, the number is only valid when it's in $$[0, M)$$.
When $$M \ll N$$, the chance to get a valid random number is very small.
For example, if $$N = 101, M = 2$$,
the probability to get a valid number is lower than $$2\%(2/101)$$.
The produced number is only valid when it's $$0$$ or $$1$$.
It's inefficient.

A simple solution is to make the number valid in $$[0, k \cdot M)$$,
where the $$k \in \mathbb{N}$$.
The $$k \cdot M$$ is the maximal multiple of $$M$$ in $$[0, N)$$.
If the number is in $$[0, k \cdot M)$$,
then we can take the remainder of the produced number divided by $$M$$
as the produced random number.

| number     | probability     | evaluated  | 
| ---------- | --------------- | ---------- |
| $$0$$      | $$\frac{1}{N}$$ | $$0$$      |
| $$1$$      | $$\frac{1}{N}$$ | $$1$$      |
| $$2$$      | $$\frac{1}{N}$$ | $$2$$      |
| $$\vdots$$ | $$\vdots$$      | $$\vdots$$ |
| $$M-1$$    | $$\frac{1}{N}$$ | $$M-1$$    |
| $$M$$      | $$\frac{1}{N}$$ | $$0$$      |
| $$M+1$$    | $$\frac{1}{N}$$ | $$1$$      |
| $$M+2$$    | $$\frac{1}{N}$$ | $$2$$      |
| $$\vdots$$ | $$\vdots$$      | $$\vdots$$ |
| $$2M-1$$   | $$\frac{1}{N}$$ | $$M-1$$    |
| $$2M$$     | $$\frac{1}{N}$$ | $$0$$      |
| $$2M+1$$   | $$\frac{1}{N}$$ | $$1$$      |
| $$2M+2$$   | $$\frac{1}{N}$$ | $$2$$      |
| $$\vdots$$ | $$\vdots$$      | $$\vdots$$ |
| $$kM-1$$   | $$\frac{1}{N}$$ | $$M-1$$    |
| $$kM$$     | $$\frac{1}{N}$$ | $$0$$      |
| $$kM+1$$   | $$\frac{1}{N}$$ | $$kM+1$$   |
| $$kM+2$$   | $$\frac{1}{N}$$ | $$kM+2$$   |
| $$\vdots$$ | $$\vdots$$      | $$\vdots$$ |
| $$N-1$$    | $$\frac{1}{N}$$ | $$N-1$$    |


| evaluated  | probability                 |
| ---------- | --------------------------- |
| $$0$$      | $$\frac{k}{N}$$             | 
| $$1$$      | $$\frac{k}{N}$$             | 
| $$2$$      | $$\frac{k}{N}$$             | 
| $$\vdots$$ | $$\vdots$$                  |
| $$M-1$$    | $$\frac{k}{N}$$             |
| others     | $$\frac{N - k \cdot M}{N}$$ |


As a result, the probability to get the number $$i$$ is


$$
\sum_{j=1}^\infty {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}
= \frac{k}{N}
+ \frac{N - k \cdot M}{N} \cdot \frac{k}{N}
+ {(\frac{N - k \cdot M}{N})}^{2} \cdot \frac{k}{N}
+ \ldots
+ {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}
$$

This is same for all number $$i$$, where $$i \in [0, M)$$.


| number     | probability                                                               |
| ---------- | ------------------------------------------------------------------------- |
| $$0$$      | $$\sum_{j=1}^\infty {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}$$ |
| $$1$$      | $$\sum_{j=1}^\infty {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}$$ |
| $$2$$      | $$\sum_{j=1}^\infty {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}$$ |
| $$\vdots$$ | $$\vdots$$                                                                |
| $$M-1$$    | $$\sum_{j=1}^\infty {(\frac{N - k \cdot M}{N})}^{j-1} \cdot \frac{k}{N}$$ |



The higher $$k$$ is, the higher probability to get a valid number.
In the above example, if $$N = 101, M = 2$$, then $$k = 50$$.
The probability to get a valid number is $$100/101$$
since the produced number is valid from $$0$$ to $$99$$.
Every even number in $$[0, 100)$$ ($$0, 2, 4, \dots, 98$$)
will be evaluated to $$0$$
and every odd number in $$[0, 100)$$ ($$1, 3, 5, \dots, 99$$)
will be evaluated to $$1$$,
since the remainders of the produced even numbers divided by $$M$$
and odd numbers divided by $$M$$ are $$0$$ and $$1$$ respectively.
That is, the probability becomes $$k$$ times.

To sum up, the algorithm is
1. Define $$K$$ to the max multiple of $$M$$ in $$N$$
2. Get a random number $$x$$ in $$[0, N)$$
3. If $$x$$ is in $$[0, K)$$, then return $$x % M$$
4. Otherwise, repeat from 2

```rs
// Get a random number in [0, s) by a random number generator in [0, b),
// where both s, b are integers and s > 1, b > 1, s <= b
fn rand_small_from_big(s: u32, b: u32) -> u32 {
    assert!(s > 1 && b > 1 && s <= b);
    // Get the max multiple of s in [0, b)
    let max = (b / s) * s;
    assert_eq!(max % s, 0);
    loop {
        // Get a random number in [0, b)
        let x = rand(b);
        // Return the random number if it is in [0, max)
        if x < max {
            return x % s;
        }
        // Repeating if the random number is in [max, b)
    }
}
```

See the live demo [here][rand_small_from_big_better].

## $$M \gt N$$

However, if $$M \gt N$$, our method above doesn't work,
since the range of $$[0, M)$$ is bigger than $$[0, N)$$ *(really? keep reading!)*.

To get a random number in $$[0, M)$$,
we need to **enlarge** the range of the numbers produced
by the given generator. How to do that ?

OK, the number of results from the generator is $$N$$ now.
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
It is how the **base-$$N$$** number works.

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
2. Define $$Y$$ to the max multiple of $$M$$ in $$N^k$$
3. Get the random number generator in $$[0, N^k)$$
4. Generate a random number in $$[0, M)$$:
    1. Get a random number $$x$$ in $$[0, N^k)$$
    2. If $$x$$ is in $$[0, Y)$$, then return $$x % M$$
    3. Otherwise, repeat from 3-1

```rs
// Get a random number in [0, b) by a random number generator in [0, s),
// where both s, b are integers and s > 1, b > 1, s <= b
fn rand_big_from_small(b: u32, s: u32) -> u32 {
    assert!(s > 1 && b > 1 && s <= b);
    // Get a minimal number p such that s^p >= b
    let p = min_pow(s, b);
    assert!(s.pow(p) >= b);
    // Get the max multiple of b in [0, s^p)
    let max = (s.pow(p) / b) * b;
    assert_eq!(max % b, 0);
    loop {
        // Get a random number in [0, s^p)
        let r = rand_pow(s, p);
        // Return the random number if it is in [0, max)
        if r < max {
            return r % b;
        }
        // Repeating if the random number is in [max, s^p)
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

See the live demo [here][rand_big_from_small_better].

### Enlarge from $$[0, 1, \ldots, N-1]$$ to $$[0, 1, \ldots, N^k-1]$$

An interesting view to look the line $$x * N + rand(N)$$ is to think
$$x$$ is a $$l$$ numbers sequence from $$a$$ to $$a + l - 1$$, $$[a, a + 1, a + 2, \ldots, a + l - 1]$$
and $$rand(N)$$ is a list from $$0$$ to $$N-1$$, $$[0, 1, 2, \ldots, N - 1]$$.
In this case, $$x * N + rand(N)$$ is something like

```rs
// x is [a, a + 1, a + 2, ..., a + l - 1]
fn enlarge(x: Vec<u32>, N: u32) -> Vec<u32> {
    let offsets: Vec<u32> = (0..N).collect();
    let mut out = Vec::new();
    for i in x.iter() {
        for j in offsets.iter() {
            out.push(N * i + j);
        }
    }
    out
}
```

What `enlarge` does is to enlarge a sequence
from $$[a, a + 1, a + 2, \ldots, a + l - 1]$$
to $$[a \cdot N, a \cdot N + 1, a \cdot N + 2, \ldots, (a + l) \cdot N  - 1)]$$.

By multiplying $$N$$ on each value in $$[a, a + 1, a + 2, \ldots, a + l - 1]$$,
the difference between $$i$$ and $$i + 1$$ is enlarged to $$N$$, where $$i \in [0, a + l - 1)$$.
The sequence becomes $$[a \cdot N, (a + 1) \cdot N, (a + 2) \cdot N, \ldots, (a + l - 1) \cdot N]$$.
By padding $$0$$, $$1, 2, \ldots , N-1$$ to the gaps between each value
in $$[a \cdot N, (a + 1) \cdot N, (a + 2) \cdot N, \ldots, (a + l - 1) \cdot N]$$,
The sequence becomes $$[a \cdot N, a \cdot N + 1, a \cdot N + 2, \ldots, (a + l) \cdot N  - 1)]$$.
This is all the values in $$[a, (a + l) \cdot N)$$. 
In other words, the sequence containing all the values in $$[a, a + l)$$
can be enlarged to a sequence containing all the values in $$[a, (a + l) \cdot N)$$
by applying `enlarge`.

If $$a$$ is $$0$$ and $$l$$ is $$N^p$$,
then we can `enlarge` a $$[0, N^p)$$-sequence to a $$[0, N^{p+1})$$-sequence.

Based on this idea, the following code

```rs
let mut x = 0;
for _ in 0..k {
    // Repeat k times
    x = x * N + rand(N);
}
```

is similar to call `k-1`-times `enlarge`

```rs
// x is [0, 1, ..., N-1]
for _ in 0..k-1 {
    // Repeat k-1 times
    x = enlarge(x, N);
}
```

to enlarge a list from $$[0, 1, ..., N-1]$$ to $$[0, 1, \ldots, N^k-1]$$.
The first `x = x * N + rand(N)` is to generate a $$[0, 1, \ldots, N-1]$$ to `x`.

See the live demo [here][enlarge].

## Compute Rand *M* from Rand *N*

In fact, the method developped for $$M \leq N$$ case
is a special case in the method for $$M \gt N$$ case.

Recall the algorithm for $$M \gt N$$ case:
1. Find a $$k$$ such that $$N^k \geq M$$
2. Define $$Y$$ to the max multiple of $$M$$ in $$N^k$$
3. Get the random number generator in $$[0, N^k)$$
4. Generate a random number in $$[0, M)$$:
    1. Get a random number $$x$$ in $$[0, N^k)$$
    2. If $$x$$ is in $$[0, Y)$$, then return $$x % M$$
    3. Otherwise, repeat from 3-1

If $$M \leq N$$, then $$k$$ is $$1$$!

Thus, the algorithm can be summarized as:
```rs
// Get a random number in [0, m) by a random number generator in [0, n)
fn rand_m_from_n(m: u32, n: u32) -> u32 {
    assert!(m > 1 && n > 1);
    // Get a minimal number p such that n^p >= m
    let p = min_pow(n, m);
    // Get the max multiple of m in [0, n^p)
    let max = (n.pow(p) / m) * m;
    loop {
        // Get a random number in [0, n^p)
        let r = rand_pow(n, p);
        // Return the random number if it is in [0, max)
        if r < max {
            return r % m;
        }
        // Repeating if the random number is in [max, n^p)
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

See the live demo [here][rand_m_from_n_better].

### If *N* is 2
To get the random number in $$[0, 2^k)$$ where $$k \in \mathbb{N}$$,
one mentionable trick is to replace
```rs
x = x * 2 + rand(2);
```
by
```rs
x = x << 1 | rand(2);
```
when $$N = 2$$.

On the other hand, we don't need to find the max multiple of $$x$$ in $$[0, 2^p)$$,
where $$p$$ is the minimal number such that $$2^p \geq x$$.
It doesn't exist! No multiple of $$x$$ is smaller than $$2^p$$.
If it exists, it is at least $$2x$$, and such that $$2x \leq 2^p$$.
However, if $$2x \leq 2^p$$, then $$x \leq 2^{p+1}$$ rather than $$x \leq 2^p$$!

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
One way to check if the distrubution of a random number generator is uniform
is to apply [*chi square test*][chi_sq_test].
The discussion can be found [here][test_uni_dist].
The [Testing a Random Number Generator][johndcook] chapter
in *John D. Cook*'s *Beautiful Testing* is also a great reference to read.

[rand_small_from_big]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=798f79143be66cecd08f81b48a97dd4b "Compute Rand([0, s)) from Rand([0, b)), where b, s are integers and b >= s"
[rand_small_from_big_better]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=b7f99315bbdfdf475658529be8dc80b8 "Compute Rand([0, s)) from Rand([0, b)), where b, s are integers and b >= s"
<!-- [rand_big_from_small]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=fb12e41948db39ebf49340a0a151529e "Compute Rand([0, b)) from Rand([0, s)), where b, s are integers and b >= s" -->
[rand_big_from_small_better]:  https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=d91510a1d8e25fe29baf0d445d8eb0f4 "Compute Rand([0, b)) from Rand([0, s)), where b, s are integers and b >= s"
<!-- [rand_m_from_n]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ad02e341de49665aeef118bdbb10aae9 "Compute Rand([0, m)) from Rand([0, n)), where m, n are integers" -->
[rand_m_from_n_better]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=d3baaff82b2a5480cdbda41bef6e2282 "Compute Rand([0, m)) from Rand([0, n)), where m, n are integers"
[rand_from_2]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=5aa2d3ae1fe8e528628ca6fafbed5b86 "Compute Rand([0, k)) from Rand([0, 1]), where m, n are integers"

[chi_sq_test]: https://en.wikipedia.org/wiki/Chi-squared_test
[test_uni_dist]: https://math.stackexchange.com/questions/2435/is-there-a-simple-test-for-uniform-distributions
[johndcook]: https://www.johndcook.com/Beautiful_Testing_ch10.pdf

[enlarge]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=34b7017dc0fb8c866b6c6a3543aac7c0