---
layout: post
title: Compute random M from Rand N
category: [Math]
tags: [Rust, Puzzles]
mathjax: true
comments: true
---

How to get a random number in [0, M) by a random number generator that randomly throws a number in [0, N).

## If *M* <= *N*

```rs
// Get a random number in [0, s) by a random number in [0, b), where both s, b
// are integers and s > 1, b > 1, s <= b
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

## If *M* > *N*

```rs
// Get a random number in [0, b) by a random number in [0, s), where both s, b
// are integers and s > 1, b > 1, s <= b
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
// Get a random number in [0, m) by a random number in [0, n)
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
// Get a random number in [0, x) by a random number in [0, 1]
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



