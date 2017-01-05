---
layout: post
title: Bubble Sort
category: Algorithm
tags: [sorting]
comments: true
mathjax: true
---
# Bubble Sort

This post series is synchronized with my book [CodePlay][codeplay]
and this post could be read [here][bub-sort].

## Idea
The basic concept is same to [_Selection Sort_][sel_sort].
The list is rearranged from minimal to maximal value
by picking the maximal(or minimal) value from the unsorted list iteratively.

In _selection sort_, the minimal element is selected after searching whole list.
On the other hand, _bubble sort_ iteratively compares two neighbor elements
and swaps the elements if the left element is greater(or less) than
the right one, from the list head to the list tail.
Therefore, the maximal, the second-maximal, ... values will be
"bubbled" up to the tail of list one by one.

Take $$L = [ 5, 3, 4, 2, 1 ]$$ as an example:

- Step 1: Move the __maximal__ value to the __last__ position of the list.
  - $$5 > 3$$ so swap them. $$L = [ 3, 5, 4, 2, 1 ]$$
  - $$5 > 4$$ so swap them. $$L = [ 3, 4, 5, 2, 1 ]$$
  - $$5 > 2$$ so swap them. $$L = [ 3, 4, 2, 5, 1 ]$$
  - $$5 > 1$$ so swap them. $$L = [ 3, 4, 2, 1, 5 ]$$
  - The maximal value $$5$$ is moved to the last of the list
- Step 2: Move the __second-maximal__ value to the __second-last__ position of the list.
  - $$3 < 4$$ so do nothing.
  - $$4 > 2$$ so swap them. $$L = [ 3, 2, 4, 1, 5 ]$$
  - $$4 > 1$$ so swap them. $$L = [ 3, 2, 1, 4, 5 ]$$
  - The second-maximal value $$4$$ is moved to the second-last of the list
- Step 3: Move the __third-maximal__ value to the __third-last__ position of the list.
  - $$3 > 2$$ so swap them. $$L = [ 2, 3, 1, 4, 5 ]$$
  - $$3 > 1$$ so swap them. $$L = [ 2, 1, 3, 4, 5 ]$$
  - The third-maximal value $$3$$ is moved to the third-last of the list
- Step 4: Move the __fourth-maximal__ value to the __fourth-last__ position of the list.
  - $$2 > 1$$ so swap them. $$L = [ 1, 2, 3, 4, 5 ]$$
  - The fourth-maximal value $$2$$ is moved to the fourth-last of the list

### Dividing one list into unsorted list and sorted list

In above example, the list is naturally partition into the sorted part
and the unsorted part.
The part contains the "bubbled" elements are sorted.
The others are unsorted.

- Step 1: Move the __maximal__ value to the __last__ position of the list.
  - This iteration ends after the comparison of last two elements(_4th_ and _5th_)
  - The maximal value $$5$$ is "bubbled" to the last(_5th_) of the list
  - $$L = [ 3, 2, 4, 1, | 5 ]$$
  - The _5th_ element is sorted part and the _1st-to-4th_ is unsorted part
- Step 2: Move the __second-maximal__ value to the __second-last__ position of the list.
  - This iteration ends after the comparison of second-last two elements(_3rd_ and _4th_)
  - The second-maximal value $$4$$ is "bubbled" to the second-last(_4th_) of the list
  - $$L = [ 3, 2, 1, | 4, 5 ]$$
  - The _4th-to-5th_ elements is sorted part and the _1st-to-3rd_ is unsorted part
- Step 3: Move the __third-maximal__ value to the __third-last__ position of the list.
  - This iteration ends after the comparison of third-last two elements(_2nd_ and _3rd_)
  - The third-maximal value $$3$$ is "bubbled" to the third-last(_3rd_) of the list
  - $$L = [ 2, 1, | 3, 4, 5 ]$$
  - The _3rd-to-5th_ elements is sorted part and the _1st-to-2nd_ is unsorted part
- Step 4: Move the __fourth-maximal__ value to the __fourth-last__ position of the list.
  - This iteration ends after the comparison of fourth-last two elements(_1st_ and _2nd_)
  - The fourth-maximal value $$2$$ is "bubbled" to the fourth-last(_2nd_) of the list
  - $$L = [ 1, | 2, 3, 4, 5 ]$$
  - The _2nd-to-5th_ elements is sorted part and the _1st_ is unsorted part
  - The last left element is definitely the __smallest__ value
  - so whole list from _1st_ to _5th_ is sorted

## Algorithm

$$
\begin{align}
& \text{BubbleSort($L$):} \\
& \space \space \space \space \text{for $i \leftarrow 1$ to $\vert L \vert - 1$:} \\
& \space \space \space \space \space \space \space \space \text{for $j \leftarrow 1$ to $\vert L \vert - i$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \text{if $L[j] > L[j+1]$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \space \space \space \space \text{swap $L[j]$ and $L[j+1]$} \\
\end{align}
$$

- At the 1st round, the 1st-last element is sorted
- At the 2nd round, the 2nd-last to 1st-last elements are sorted
- At the 3rd round, the 3rd-last to 1st-last elements are sorted
- At the $$k$$ round, the $$k$$-last to 1st-last elements are sorted

### Proof

#### Proof by mathematical induction

##### Lemma 1
$$
\begin{align}
& \text{for $j \leftarrow 1$ to $\vert L \vert - 1$:} \\
& \space \space \space \space \text{if $L[j] > L[j+1]$:} \\
& \space \space \space \space \space \space \space \space \text{swap $L[j]$ and $L[j+1]$} \\
\end{align}
$$

> Given a list $$L$$ with $$N$$ elements, where $$N = \vert L \vert > 0$$,
> the maximal element of $$L$$ will be $$L[N]$$(the last element).

- Base step: When $$N = 1$$, the assumption obviously holds
- Induction Hypothesis: Assume the hypothesis holds when $$N = k(j \leftarrow 1 \text{ to } k-1)$$
- Induction Step: when $$N = k + 1$$
  - After the iteration of $$j = k - 1$$,
    the list is divided into two parts: $$L[1...k]$$ and $$L[k+1]$$
  - From above hypothesis, the maximal value in $$L[1...k]$$ is $$L[k]$$
  - When $$j = k$$:
    - if $$L[k] \leq L[k+1]$$, then the maximal element is $$L[k+1]$$,
      so the hypothesis still holds
    - if $$L[k] > L[k+1]$$, they will be swapped.
      After then, the maximal element will be $$L[k+1]$$,
      so the hypothesis still holds

##### Lemma 2
$$
\begin{align}
& \text{for $i \leftarrow 1$ to $\vert L \vert - 1$:} \\
& \space \space \space \space \text{for $j \leftarrow 1$ to $\vert L \vert - i$:} \\
& \space \space \space \space \space \space \space \space \text{if $L[j] > L[j+1]$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \text{swap $L[j]$ and $L[j+1]$} \\
\end{align}
$$

> After the $$i$$ iteration,
> $$L[N - i + 1]$$ will be the $$i$$th largest element of $$L[1...N]$$

- Base step: When $$i = 1$$, the assumption holds because _lemma 1_ is true
- Induction Hypothesis: Assume hypothesis holds when $$i = k$$
- Induction Step: When $$i = k + 1$$
  - The goal is to prove that the $$L[N - k]$$ is
    the $$(k + 1)$$th largest element of $$L[1...N]$$
  - After the iteration $$i = k$$, $$L[N-k+1]$$ is the $$k$$th
    largest element of $$L[1...N]$$
  - We can divide the list into $$L[1...N-k]$$ and $$L[N-k+1...N]$$
    - The list $$L[N-k+1...N]$$ contains the picked
      $$k$$th, $$(k-1)$$th, .... , $$1$$st largest elements
    - The list $$L[1...N-k]$$ is the unselected and unsorted list
  - By applying the _lemma 1_ to $$L[1...N-k]$$,
    the maximal element of $$L[1...N-k]$$ will be $$L[N-k]$$
    after all the iterations for $$1 \leq j \leq N - k$$.
  - $$L[N-k]$$ is the $$(k+1)$$th selected maximal element
  - so $$L[N - (k+1) + 1] = L[N-k]$$ is the $$(k+1)$$th largest element of $$L[1...N]$$

##### Conclusion

By _lemma 2_
- After $$i = 1$$, $$L[N]$$ is the $$1$$st largest element of $$L[1...N]$$
- After $$i = 2$$, $$L[N - 1]$$ is the $$2$$nd largest element of $$L[1...N]$$
- After $$i = 3$$, $$L[N - 2]$$ is the $$3$$rd largest element of $$L[1...N]$$
- ...
- After $$i = k$$, $$L[N - k + 1]$$ is the $$k$$th largest element of $$L[1...N]$$
- After $$i = k+1$$, $$L[N - k]$$ is the $$(k+1)$$th largest element of $$L[1...N]$$
- ...
- After $$i = N-2$$, $$L[3]$$ is the $$N-2$$th largest element of $$L[1...N]$$
- After $$i = N-1$$, $$L[2]$$ is the $$N-1$$th largest element of $$L[1...N]$$
- After $$i = N$$, $$L[1]$$ is the $$N$$th largest element of $$L[1...N]$$

Thus, the list $L[1...N]$ is sorted by the order that $$L[1] \leq L[2] \leq L[3] \leq ... \leq L[N-1] \leq L[N]$$.

## Complexity

Let $$N = \vert L \vert$$ denote the length of list $$L$$.

| $$i$$   | iterations for $$j$$ |
| ------- | -------------------- |
| $$1$$   | $$j \in [1, N-1]$$   |
| $$2$$   | $$j \in [1, N-2]$$   |
| ...     | ...                  |
| $$N-2$$ | $$j \in [1, 2]$$     |
| $$N-1$$ | $$j = 1$$            |


The total of all iterations of _BubbleSort($$L$$)_ is tracked in above table
and its sum is:

$$
\begin{align}
(N - 1) + (N - 2) + ... + 2 + 1
&= \frac{ N \cdot (N - 1) }{ 2 } \\
&= \frac{ 1 }{ 2 } \cdot N^2 - \frac{ 1 }{ 2 } N
\end{align}
$$

Therefore, the complexity is $$\mathcal{O}(N^2)$$.

## Implementation

See the files on [gist here][gist].

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting.h" data-gist-line="1-14, 19-20, 25"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting.cpp" data-gist-line="1, 5-20, 109-130"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting_test.cpp" data-gist-line="2-37, 45-58, 73-79, 100-101"></code>


The above gist files are imported by [gist-embed][gist-embed].

[gist]: https://gist.github.com/ChunMinChang/dee9f3bd2ceab69726373ae006016edb "simple_sort"
[gist-embed]: https://github.com/blairvanderhoof/gist-embed "gist-embed"

<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/gist-embed/2.4/gist-embed.min.js"></script>


[codeplay]: https://www.gitbook.com/book/chunminchang/codeplay/details "CodePlay"
[bub-sort]: https://chunminchang.gitbooks.io/codeplay/content/sorting/bubble_sort.html "Bubble Sort"
[sel_sort]: https://chunminchang.gitbooks.io/codeplay/content/sorting/selection_sort.html "Selection Sort"
