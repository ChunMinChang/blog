---
layout: post
title: Selection Sort
category: Algorithm
tags: [sorting]
comments: true
mathjax: true
---
# Selection sort

This post series is synchronized with my book [CodePlay][codeplay]
and this post could be read [here][sel-sort].

## Idea
The concept is quite straight.
If we could get the minimal value from list __one by one__,
then we could re-arrange the list from minimal to maximal values.

Imagine we have two lists $$L$$ and $$L_{sorted}$$,
the $$L$$ is a list contains several items with comparable values and
the $$L_{sorted}$$ is a sorted list of $$L$$.
At first, $$L_{sorted} = [ ]$$ is empty.

Take $$L = [ 5, 3, 1, 2, 3 ]$$ as an example:

- At the first round, we get $$1$$ as minimal value,
  so we move it into $$L_{sorted}$$. Now,
  - $$L_{sorted} = [ 1 ]$$
  - $$L = [ 5, 3, 2, 3 ]$$
- At the second round, we get $$2$$ as minimal value, so
  - $$L_{sorted} = [ 1, 2 ]$$
  - $$L = [ 5, 3, 3 ]$$
- Next, $$3$$ is picked and moved from $$L$$ to $$L_{sorted}$$, so
  - $$L_{sorted} = [ 1, 2, 3 ]$$
  - $$L = [ 5, 3 ]$$
- Then, the current minimal value $$3$$ is moved from $$L$$ to $$L_{sorted}$$, so
  - $$L_{sorted} = [ 1, 2, 3, 3 ]$$
  - $$L = [ 5 ]$$
- Finally, $$5$$ is moved into $$L_{sorted}$$, so
  - $$L_{sorted} = [ 1, 2, 3, 3, 5 ]$$
  - $$L = [ ]$$

See! the idea is quite simple.
In the same way, to sort the list from maximal to minimal values,
the only different is to pick the maximal value from list each round
instead of minimal value.

### How to get minimal(or maximal) value
The way to get minimal(or maximal) items in $$L$$
is to linearly search the whole list:
$$
\begin{align}
& \text{Min($L$):} \\
& \space \space \space \space min = L[1] \\
& \space \space \space \space \text{for $i \leftarrow 1$ to $N$:} \\
& \space \space \space \space \space \space \space \space \text{if $L[i] < min$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space min = L[i] \\
\end{align}
$$
or
$$
\begin{align}
& \text{Max($L$):} \\
& \space \space \space \space max = L[1] \\
& \space \space \space \space \text{for $i \leftarrow 1$ to $N$:} \\
& \space \space \space \space \space \space \space \space \text{if $L[i] > max$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space max = L[i] \\
\end{align}
$$
, where $$L[i]$$ is the $$i$$th element in the list $$L$$
and $$N$$ is the length of $$L$$.

### Divide one list into unsorted list and sorted list
In implementation, we usually divide the source list,
which needs to be sorted, into two parts. One is sorted, the other is unsorted.
This is better for memory usage than
creating another list to put the sorted results.

That is, if we have a source list $$L = [ 5, 3, 1, 2, 3 ]$$,
it will be divided into $$L_{sorted}$$ and $$L_{unsorted}$$.
They are initialized to $$[]$$ and $$L$$ respectively,
so $$L = L_{sorted} \cup L_{unsorted} = [ ] \cup [ 5, 3, 1, 2, 3 ]$$.

- In the first round, $$1$$ is picked and moved
  from $$L_{unsorted}$$ to $$L_{sorted}$$, so
  - $$L = L_{sorted} \cup L_{unsorted} = [ 1 ] \cup [ 5, 3, 2, 3 ] = [1 \vert 5, 3, 2, 3]$$
- In the second round, $$2$$ is picked and moved
  from $$L_{unsorted}$$ to $$L_{sorted}$$, so
  - $$L = L_{sorted} \cup L_{unsorted} = [ 1, 2 ] \cup [ 5, 3, 3 ] = [1, 2 \vert 5, 3, 3]$$
- Next, $$3$$ is picked, so
  - $$L = L_{sorted} \cup L_{unsorted} = [ 1, 2, 3 ] \cup [ 5, 3 ] = [1, 2, 3 \vert 5, 3]$$
- Then, another $$3$$ is picked, so
  - $$L = L_{sorted} \cup L_{unsorted} = [ 1, 2, 3, 3 ] \cup [ 5 ] = [1, 2, 3, 3 \vert 5]$$
- Finally, $$5$$ is picked, so
  - $$L = L_{sorted} \cup L_{unsorted} = [ 1, 2, 3, 3, 5 ] \cup [ ] = [1, 2, 3, 3, 5]$$

## Algorithm
$$
\begin{align}
& \text{SelectionSort($L$):} \\
& \space \space \space \space \text{for $i \leftarrow 1$ to $\vert L \vert - 1$:} \\
& \space \space \space \space \space \space \space \space m \leftarrow i \\
& \space \space \space \space \space \space \space \space \text{for $j \leftarrow i + 1$ to $\vert L \vert$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \text{if $L[j] < L[m]$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \space \space \space \space m = j \\
& \space \space \space \space \space \space \space \space \text{swap $L[i]$ and $L[m]$} \\
\end{align}
$$

The above method will divide $$L$$ into two parts.

- $$L[1...i-1] = L_{sorted}$$ is sorted
- $$L[i...N] = L_{unsorted}$$ is unsorted,
- where $$N = \vert L \vert$$ is the length of $$L$$

The following are step by step explanation:

- When $$i = 1$$
  - $$L_{sorted} = []$$ and $$L_{unsorted} = L[1...N]$$
  - we need to find the minimal element in list $$L[1...N]$$
  - We use a value $$m$$ to track the __index__ of the minimal element
  - where $$m$$ is initialized to $$1$$
  - $$m$$ will be updated to $$j$$, where $$2 \leq j \leq N$$, if $$L[j] < L[m]$$
  - Repeatedly above instruction from $$j = 2$$ to $$N$$(searching whole $$L_{unsorted}$$),
    $$L[m]$$ would be the minimal value in $$L[1...N]$$
  - swap $$L[m]$$ and $$L[i = 1]$$
  - then $$L[1]$$ now can be considered as $$L_{sorted}$$
  - so $$L_{unsorted} = L[2...N]$$
- When $$i = 2$$
  - $$L_{sorted} = L[1]$$ and $$L_{unsorted} = L[2...N]$$
  - we need to find the minimal element in list $$L[2...N]$$
  - Same as above, $$m$$ is used to keep tracking the index of the minimal
  element in $$L_{unsorted}$$
  - where $$m$$ is initialized to $$2$$
  - After searching the whole $$L_{unsorted}$$,
    $$L[m]$$ would be the minimal value in $$L[2...N]$$
  - We can put $$L[m]$$ into the $$L_{sorted}$$ by swapping the $$L[m]$$ and $$L[i = 2]$$
  - Thus, $$L_{sorted} = L[1...2]$$ and $$L_{unsorted} = L[3...N]$$
- When $$i = k$$
  - $$L_{sorted} = [1...k-1]$$ and $$L_{unsorted} = L[k...N]$$
  - $$m$$ is initialized to $$k$$
  - After searching the whole $$L_{unsorted}$$,
    $$L[m]$$ would be the minimal value in $$L[k...N]$$
  - Swapping $$L[m]$$ and $$L[i = k]$$ would put $$L[m]$$ into $$L_{sorted}$$
  - Then, $$L_{sorted} = L[1...k]$$ and $$L_{unsorted} = L[k+1...N]$$
- When $$i = N - 1$$(final round)
  - $$L_{sorted} = [1...N-2]$$ and $$L_{unsorted} = L[N-1...N]$$
  - $$m$$ is initialized to $$N-1$$
  - Pick a smaller one between $$L[N-1]$$ and $$L[N]$$
  - and put it into the $$L_{sorted}$$ like above(by swapping with $$L[m]$$)
  - Then $$L_{sorted} = [1...N-1]$$ and $$L_{unsorted} = L[N]$$
  - The left one(now is $$L[N]$$) is definitely the __maximal__ item
    in $$L[1...N]$$, so we don't need to do anything

### Proof

#### Proof by contradiction

- Assume this method can __not__ give us an ordered list
- so it exists one $$L[p] > L[q]$$, where $$p < q$$, in the result list $$L$$
- Before the result is computed, the unsorted list could be
  $$L[..p..q..]$$ or $$L[..q..p..]$$
- It means that $$L[p]$$ is picked __before__ $$L[q]$$
  because $$p < q$$ in the result list $$L$$
- It means $$L[p] < L[q]$$ and it is contradictory to the assumption
- Thus, the assumption is wrong. This method will give us an ordered list.

## Complexity
We need to search whole $$L_{unsorted}$$ to find a minimal(or maximal) item.
Suppose $$\vert L_{unsorted} \vert = N$$ at first.
(the length of $$L_{unsorted}$$ is $$N$$).

At the first round, we need to search $$N$$ items
to find the minimal(or maximal) item and move it into $$L_{sorted}$$.
After then, $$\vert L_{unsorted} \vert = N - 1$$.

At the second round, whole $$N - 1$$ items in $$L_{unsorted}$$ would be counted
to find the minimal(or maximal) one.
After the picked one is moved into $$L_{sorted}$$,
the size of $$L_{unsorted}$$ is reduced to $$\vert L_{unsorted} \vert = N - 2$$.

The procedure keep working until the list $$L_{unsorted}$$ is empty
($$\vert L_{unsorted} \vert = 0$$).
Thus, we need to search
$$
\begin{align}
N + (N - 1) + (N - 2) + .... + 1
&= \frac{ N \cdot (N + 1) }{ 2 } \\
&= \frac{ 1 }{ 2 } \cdot N^2 + \frac{ 1 }{ 2 } N
\end{align}
$$
times to move all the items into $$L_{sorted}$$.
Therfore, the complexity is $$\mathcal{O}(N^2)$$,
where the $$N$$ is the length of the list $$L$$.

## Implementation
See the files on [gist here][gist].

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="simple_sort.h" data-gist-line="1-16, 25"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="simple_sort.cpp" data-gist-line="1, 5-20, 37-73"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="simple_test.cpp" data-gist-line="2-37, 45-64, 100-101"></code>

<!-- ### General version with C++ ```template``` -->

The above gist files are imported by [gist-embed][gist-embed].


[gist]: https://gist.github.com/ChunMinChang/dee9f3bd2ceab69726373ae006016edb "simple_sort"
[gist-embed]: https://github.com/blairvanderhoof/gist-embed "gist-embed"

<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/gist-embed/2.4/gist-embed.min.js"></script>


[codeplay]: https://www.gitbook.com/book/chunminchang/codeplay/details "CodePlay"
[sel-sort]: https://chunminchang.gitbooks.io/codeplay/content/sort/selection_sort.html "Selection Sort"
