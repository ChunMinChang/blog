---
layout: post
title: Running Average
category: [Common, Math]
tags: [Firefox]
mathjax: true
comments: true
---

Preventing overflow is essential
while calculating the average of a long series of data.
One simplest method to avoid the problem is *running average*
(or *moving average*, *rolling average*, *incremental mean*).

## Problem
Suppose we have data $$x_1, x_2, \cdots, x_n$$.
The $$sum_{n+1}$$ will overflow when the new data $$x_{n+1}$$ is counted,
where $$sum_i = x_1 + x_2 + \cdots + x_i$$.
At that time, $$\mu_{n+1} = \frac{sum_{n+1}}{n+1}$$ will be wrong,
where $$\mu_i$$ is the average of $$x_1, x_2, \cdots, x_i$$,
since $$sum_{n+1}$$ is an overflowed value.

How could we solve the problem?

## Naive Approach

```c
uint64_t count = 0;
uint64_t sum = 0;

void Add(uint64_t data)
{
  if (sum + data < sum) { // Overflow!
    sum = sum / 2;
    count = count / 2;
  }

  sum += data;
  ++count;
}

double GetAverage()
{
  return static_cast<double>(sum) / count;
}
```

The above approach to prevent overflow
is extracted from [Gecko MP3Demuxer][MP3Demuxer],
but it's **wrong** in most cases
(it's only correct when every incoming data is same).
For example, if we have $$x_1 = 10, x_2 = 20$$
and $$x_3 = 90$$ will lead to overflow to the sum of $$x_i$$
so the average computed by above approach will be $$\frac{(10+20)/2 + 90}{2/2 + 1} = 52.5$$,
which is different from $$\frac{10 + 20 + 90}{3} = 40$$.
I cannot believe this incorrect code
lives in Firefox [more than 3 years](https://bugzilla.mozilla.org/show_bug.cgi?id=1093815).
Fortunately, the overflow happens once in a blue moon
so it's not too much trouble.

### Proof
We can formally prove the above approach is incorrect.
Let $$sum_{k}$$ and $$\mu_k$$ be the **sum** and **arithmetic mean**
of $$x_1, x_2, \cdots, x_k$$ respectively, and $$E_k$$ be the estimated average
from above approach.

Since the new incoming data $$x_{n+1}$$ will cause overflow to $$sum_{n+1}$$, so

$$
E_{n+1} = \frac{p + x_{n+1}}{q + 1}
$$
, where
$$
p = \frac{sum_{n}}{2} = \frac{x_1 + x_2 + \cdots + x_n}{2}
$$
and
$$
q = \frac{n}{2}
$$

As a result,

$$
\begin{align}
E_{n+1}
&= \frac{p + x_{n+1}}{q + 1} \\
&= \frac{\frac{x_1 + x_2 + \cdots + x_n}{2} + x_{n+1}}{\frac{n}{2} + 1} \\
&= \frac{\frac{x_1 + x_2 + \cdots + x_n + 2 \cdot x_{n+1}}{2}}{\frac{n + 2}{2}} \\
&= \frac{x_1 + x_2 + \cdots + x_n + 2 \cdot x_{n+1}}{n+2}
\end{align}
$$

We could compare $$E_{n+1}$$ and $$\mu_{n+1}$$ to see if they are equal:

$$
\begin{align}
\mu_{n+1}
&= \frac{x_1 + x_2 + \cdots + x_n + x_{n+1}}{n+1} \\
&= \frac{(n+2) \cdot (x_1 + x_2 + \cdots + x_n + x_{n+1})}{(n+1) \cdot (n+2)} \\
&= \frac{(n+1) \cdot (x_1 + x_2 + \cdots + x_n + x_{n+1}) + (x_1 + x_2 + \cdots + x_n + x_{n+1})}{(n+1) \cdot (n+2)}
\\
E_{n+1}
&= \frac{x_1 + x_2 + \cdots + x_n + 2 \cdot x_{n+1}}{n+2} \\
&= \frac{(x_1 + x_2 + \cdots + x_n + x_{n+1}) + x_{n+1}}{n+2} \\
&= \frac{(n+1) \cdot ((x_1 + x_2 + \cdots + x_n + x_{n+1}) + x_{n+1})}{(n+1) \cdot (n+2)} \\
&= \frac{(n+1) \cdot (x_1 + x_2 + \cdots + x_n + x_{n+1}) + (n+1) \cdot x_{n+1}}{(n+1) \cdot (n+2)} \\
&= \frac{(n+1) \cdot (x_1 + x_2 + \cdots + x_n + x_{n+1}) + (\overbrace{x_{n+1} + \cdots + x_{n+1}}^{n+1}))}{(n+1) \cdot (n+2)}
\\
\mu_{n+1} - E_{n+1}
&= \frac{ (x_1 + x_2 + \cdots + x_n + x_{n+1}) - (\overbrace{x_{n+1} + \cdots + x_{n+1}}^{n+1}))}{(n+1) \cdot (n+2)} \\
&= \frac{ (x_1 - x_{n+1}) + (x_2 - x_{n+1}) + \cdots +  (x_n - x_{n+1}) + (x_{n+1} - x_{n+1})}{(n+1) \cdot (n+2)} \\
&= \frac{ (x_1 - x_{n+1}) + (x_2 - x_{n+1}) + \cdots +  (x_n - x_{n+1}) }{(n+1) \cdot (n+2)}
\end{align}
$$

Thus, we can clearly see that
$$\mu_{n+1}$$ will be equal to $$E_{n+1}$$
**only** when $$x_1 = x_2 = \cdots = x_n = x_{n+1}$$.

## Running Average

In fact, the average can be calculated
without using the overflowed $$sum_{n+1}$$.
Let $$\mu_n$$ be the mean of $$x_1, x_2, \cdots, x_n$$,
we can get $$\mu_n$$ by $$\mu_{n-1}$$:

$$
\begin{align}
\mu_n
&= \frac{ \mu_{n-1} \cdot (n-1) + x_n }{ n } \\
&= \frac{ n \cdot \mu_{n-1} + x_n - \mu_{n-1}}{ n } \\
&= \mu_{n-1} + \frac{ x_n - \mu_{n-1} }{ n }
\end{align}
$$

### Sample code

On this ground, we could correct the [code][gist] into:

```c
double average = 0;
uint64_t count = 0;
void Add(uint64_t data)
{
  average += (data - average) / ++count;
}

double GetAverage()
{
  return average;
}
```

or

```c
double UpdateAverage(uint64_t data)
{
  static double average = 0;
  static uint64_t count = 0;
  average += (data - average) / ++count;
  return average;
}
```

or

```cpp
class Averager
{
public:
  Averager()
    : average(0)
    , count(0)
  {}

  ~Averager() {};

  void Add(uint64_t data) { average += (data - average) / ++count; }

  double GetAverage() { return average; }

private:
  double average;
  uint64_t count;
}
```

## References

[Bug 1423834][b1423834] has been filed for this problem
when I found this code in [MP3Demuxer][MP3Demuxer].
You could find more detail there.

The following links are some related resources:
- [Moving average on Wiki](https://en.wikipedia.org/wiki/Moving_average)
- [Incremental averageing](https://math.stackexchange.com/questions/106700/incremental-averageing)
- [Prevent long running averaging from overflow?](https://stackoverflow.com/questions/3316261/prevent-long-running-averaging-from-overflow)

[MP3Demuxer]: https://searchfox.org/mozilla-central/rev/f5f1c3f294f89cfd242c3af9eb2c40d19d5e04e7/dom/media/mp3/MP3Demuxer.cpp#709-715,720,728,756-760 "Wrong calculation for mp3 time length as we prevent overflow"
[b1423834]: https://bugzilla.mozilla.org/show_bug.cgi?id=1423834 "Bug 1423834 - Wrong calculation for mp3 time length as we prevent overflow"
[gist]: https://gist.github.com/ChunMinChang/a1d7533859dba59a1701d1d42c29bf82 "Calculating average without sum"