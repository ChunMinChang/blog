---
layout: post
title: Insertion Sort
category: Algorithm
tags: [sorting]
comments: true
mathjax: true
---
# Insertion sort

This post series is synchronized with my book [CodePlay][codeplay]
and this post could be read [here][ins-sort].

## Idea
The basic concept is similar to [_Selection Sort_][sel_sort].
Considering there are two lists. One is already sorted,
and the other is unsorted, denoted $$L_{sorted}$$ and $$L_{unsorted}$$ respectively.
The key idea is to pick the element from $$L_{unsorted}$$ one by one
and then __insert__ them into the correct positions of $$L_{sorted}$$.
Suppose we have $$L_{sorted} = [3, 8, 34]$$
and $$L_{unsorted} = [23, 2, 67, 34, 97]$$:

- Step 1
  - Pick $$23$$ (which is the first element) from $$L_{unsorted}$$,
  and insert it into $$L_{sorted}$$
  - Find a position in $$L_{sorted}$$ such that
  __all elements before it is less than or equal to $$23$$
  and all elements after it is greater than $$23$$__
  - Start comparing it from the __last(maximal)__ element
  to the __first(minimal)__ one in $$L_{sorted}$$
  (Or you can do same thing from the first element to the last one)
  - $$34$$ is greater than $$23$$, so we keep moving
  - Next, we found that $$8$$ is less than $$23$$
  - A-ha! $$23$$ should be inserted between $$8$$ and $$34$$
  - The $$L_{sorted}$$ and $$L_{unsorted}$$ are updated to $$[3, 8, 23, 34]$$
  and $$[2, 67, 34, 97]$$ respectively.
- Step 2
  - Pick the current first element of $$L_{unsorted}$$, $$2$$,
  and insert it into $$L_{sorted}$$
  - Same as the previous step, we start comparing $$2$$ from the maximal element
  of $$L_{sorted}$$ to find the position to insert
  - $$34$$ is obviously larger than $$2$$, so we should keep moving
  - In this step, we can not find any element less than or equal to $$2$$ after
  the all elements in $$L_{sorted}$$ are checked
  - Thus, the $$2$$ is the minimal value among these elements
  - We should put $$2$$ as the first element in $$L_{sorted}$$
  - The $$L_{sorted}$$ and $$L_{unsorted}$$ are updated to $$[2, 3, 8, 23, 34]$$
  and $$[67, 34, 97]$$ respectively
- Step 3
  - pick the current first element of $$L_{unsorted}$$, $$67$$,
  and then insert it into $$L_{sorted}$$
  - Start comparing $$67$$ with $$34$$, we found $$67$$ is greater
  - It means that $$67$$ is the maximal value among these elements
  - Therefore, $$67$$ should be inserted at the last position of $$L_{sorted}$$
  - The $$L_{sorted}$$ and $$L_{unsorted}$$ are updated to
  $$[2, 3, 8, 23, 34, 67]$$ and $$[34, 97]$$ respectively
- Step 4
  - $$34$$ is picked to compare with the elements in $$L_{sorted}$$.
  - $$67$$ is greater than $$34$$, so go next
  - $$34$$ is equal to $$34$$, so we stop here
  - The picked $$34$$ should be inserted between the existed $$34$$ and $$67$$
  - so the $$L_{sorted}$$ and $$L_{unsorted}$$ are updated to
  $$[2, 3, 8, 23, 34, 34, 67]$$ and $$[97]$$ respectively.
- Step 5
  - $$97$$ is picked to insert.
  - $$97$$ is greater than $$67$$,
  - so it should be put to the last position of $$L_{sorted}$$
  - Finally, $$L_{unsorted}$$ is empty
  and $$L_{sorted} = [2, 3, 8, 23, 34, 34, 67, 97]$$.


### How to find the inserted position

We can use the following method to find the __first__ element
whose value is __less than or equal to__ the picked element:
$$
\begin{align}
& \text{Position($L, x$):} \\
& \space \space \space \space i \leftarrow N\\
& \space \space \space \space \text{while $i > 0$ and $L[i] > x$:} \\
& \space \space \space \space \space \space \space \space i \leftarrow i - 1 \\
& \space \space \space \space \text{return} \space i \\
\end{align}
$$
,where $$x$$ is the element needs to be inserted,
$$L[i]$$ is the $$i$$th element in the sorted list $$L$$,
and $$N = \vert L \vert$$ is the length of $$L$$.

After getting the position $$p = Position(L, x)$$ given the element $$x$$,
we need to insert $$x$$ between $$L[p]$$ and $$L[p+1]$$.
(If $$p = 0$$, then we insert $$x$$ as the first element $$L[1]$$.
If $$p = N$$, then we insert $$x$$ as the last element $$L[p + 1]$$.)

### Divide one list into unsorted list and sorted list

In implementation, we usually divide the source list $$L$$ into two parts.
One is sorted, the other is unsorted.
They are denoted $$L_{sorted}$$ and $$L_{unsorted}$$ respectively.
This is better for memory usage than
creating another list to put the sorted results.

Suppose we have $$L = [73, 24, 37, 9, 97, 29] = L_{sorted} \cup L_{unsorted}$$,
where $$L_{sorted}$$ and $$L_{unsorted}$$ are initialized to $$[]$$
and $$[73, 24, 37, 9, 97, 29]$$.

- First round
  - $$73$$ is picked, but there is nothing could be compared
  - so we just put it into $$L_{sorted}$$
  - $$L_{sorted} = [73]$$ and $$L_{unsorted} = [24, 37, 9, 97, 29]$$
  - now $$L = L_{sorted} \cup L_{unsorted} = [73 \vert 24, 37, 9, 97, 29]$$
- Second round
  - $$24$$ is picked and $$p = Position(L_{sorted}, 24) = 0$$
  - so, we should insert $$24$$ as the __first__ element and update lists
  - then $$L_{sorted} = [24, 73]$$ and $$L_{unsorted} = [37, 9, 97, 29]$$
  - now $$L = L_{sorted} \cup L_{unsorted} = [24, 73 \vert 37, 9, 97, 29]$$
- Third round
  - $$37$$ is picked and $$p = Position(L_{sorted}, 37) = 1$$
  - so we should insert $$37$$ between $$L[p] = L[1] = 24$$ and $$L[p + 1] = L[2] = 73$$
  - then $$L_{sorted} = [24, 37, 73]$$ and $$L_{unsorted} = [9, 97, 29]$$
  - now $$L = L_{sorted} \cup L_{unsorted} = [24, 37, 73 \vert 9, 97, 29]$$
- Fourth round
  - $$9$$ is picked and $$p = Position(L_{sorted}, 9) = 0$$
  - Thus, $$L_{sorted}, L_{unsorted}$$ are updated to
    $$[9, 24, 37, 73]$$ and $$[97, 29]$$.
  - now $$L = L_{sorted} \cup L_{unsorted} = [9, 24, 37, 73 \vert 97, 29]$$
- Fifth round
  - $$97$$ is picked and $$p = Position(L_{sorted}, 97) = 4 = \vert L_{sorted} \vert$$
  - so we should put $$97$$ as the __last__ element of the $$L_{sorted}$$
  - then $$L_{sorted} = [9, 24, 37, 73, 97]$$ and $$L_{unsorted} = [29]$$
  - now $$L = L_{sorted} \cup L_{unsorted} = [9, 24, 37, 73, 97 \vert 29]$$.
- Final round
  - $$29$$ is picked and $$p = Position(L_{sorted}, 29) = 2$$
  - so we should insert $$29$$ between $$L[p] = L[2] = 24$$ and $$L[p + 1] = L[3] = 37$$
  - then $$L_{sorted} = [9, 24, 29, 37, 73, 97]$$ and $$L_{unsorted} = []$$ is empty
  - now $$L = L_{sorted} \cup L_{unsorted} = [9, 24, 29, 37, 73, 97]$$.

## Algorithm
$$
\begin{align}
& \text{InsertionSort($L$):} \\
& \space \space \space \space \text{for $i \leftarrow 2$ to $\vert L \vert$:} \\
& \space \space \space \space \space \space \space \space j \leftarrow i\\
& \space \space \space \space \space \space \space \space \text{while $j > 1$ and $L[j-1] > L[j]$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space \text{swap $L[j-1]$ and $L[j]$} \\
& \space \space \space \space \space \space \space \space \space \space \space \space j = j - 1 \\
\end{align}
$$

The above method will divide $$L$$ into two parts.
$$L[1...i-1] = L_{sorted}$$ is sorted, and $$L[i...N] = L_{unsorted}$$ is unsorted,
where $$N = \vert L \vert$$ is the length of $$L$$.
The $$L[i]$$ will be picked to insert into $$L_{sorted}$$ iteratively.

- When $$i = 2$$
  - $$L_{sorted} = L[1]$$ and $$L_{unsorted} = L[2...N]$$
  - The goal in this round is to insert the $$L[2]$$ into $$L_{sorted}$$
  - The $$L[2]$$ is picked and compare with $$L[1]$$
  - If $$L[2] < L[1]$$, then we swap them
  - Otherwise, do nothing
  - Then, $$L_{sorted} = L[1...2]$$ is sorted and $$L_{unsorted} = L[3...N]$$
- When $$i = 3$$
  - $$L_{sorted} = L[1...2]$$ and $$L_{unsorted} = L[3...N]$$
  - The goal in this round is to insert the $$L[3]$$ into $$L_{sorted}$$
  - The $$L[3]$$ is picked
  - If $$L[3] >= L[2]$$, it means that $$L[1...3]$$ is sorted, so we don't need to do anything
  - Otherwise($$L[3] < L[2]$$), swap $$L[3]$$ and $$L[2]$$
  and check whether it needs to swap again if $$L[2] < L[1]$$
  - After finishing checking, $$L_{sorted} = L[1...3]$$ is sorted
  and $$L_{unsorted} = L[4...N]$$
- When $$i = k$$
  - $$L_{sorted} = L[1...k-1]$$ and $$L_{unsorted} = L[k...N]$$
  - The goal in this round is to insert the $$L[k]$$ into $$L_{sorted}$$
  - The $$L[k]$$ is picked to compare with the elements one by one in $$L_{sorted}$$,
  from the maximal($$L[k-1]$$) to minimal item($$L[1]$$), to find a place to insert
  - After finishing checking, $$L_{sorted} = L[1...k]$$ is sorted
  and $$L_{unsorted}$$ is $$L[k+1...N]$$
- When $$i = N$$
  - $$L_{sorted} = L[1...N-1]$$ and $$L_{unsorted} = L[N]$$
  - The goal in this round is to insert the $$L[N]$$ into $$L_{sorted}$$
  - In the same way, the $$L[1...N]$$ is sorted after finishing the procedure
  - so $$L_{sorted}$$ is updated to $$L[1...N]$$ and $$L_{unsorted} = []$$ is empty


### Another method without swapping
$$
\begin{align}
& \text{InsertionSort($L$):} \\
& \space \space \space \space \text{for $i \leftarrow 2$ to $\vert L \vert$:} \\
& \space \space \space \space \space \space \space \space c \leftarrow L[i] \\
& \space \space \space \space \space \space \space \space j \leftarrow i \\
& \space \space \space \space \space \space \space \space \text{while $j > 1$ and $L[j-1] > c$:} \\
& \space \space \space \space \space \space \space \space \space \space \space \space L[j] = L[j-1] \\
& \space \space \space \space \space \space \space \space \space \space \space \space j = j - 1 \\
& \space \space \space \space \space \space \space \space L[j] = c \\
\end{align}
$$

### Proof

#### Proof by mathematical induction

> After each iteration for $$i$$ in $$InsertionSort$$,
  the $$L[1...i]$$ is sorted array.

We need to prove this statement is true.

- when $$i = 2$$:
  - Same as the above explanation
  - The assumption is hold
- when $$i = k$$:
  - Assume the statement is hold when $$i = k$$
  - $$L[1...k]$$ is sorted array
- when $$i = k + 1$$
  - If $$L[k + 1] > L[k]$$, then $$L[1...k + 1]$$ is naturally sorted
  so the proof is done
  - Otherwise, the $$L[k + 1]$$ is swapped with $$L[k]$$
  - Now $$L[1...k-1]$$ is sorted and $$L[k + 1] > L[k]$$(after swapping!)
  - Next, we apply this algorithm to $$L = L[1...k-1] \cup L[k]$$ and now $$i = k$$
  - The statement is hold when $$i = k$$,
  so $$L[1...k]$$ is sorted after applying the algorithm
  - Now $$L[1...k]$$ is sorted and $$L[k + 1] > L[k]$$, so the proof is done

## Complexity
The time complexity depends on the speed to find the inserted position.
The more iterations to find the value of $$Position(L, x)$$ need,
the more time it takes.
The worst case is that we need to go through whole $$L_{sorted}$$ to find correct
positions to insert. It happens when the list is arranged from maximal to
minimal values(e.g.,$$[5, 4, 3, 2, 1]$$).
In this case, if the length of list is $$N$$, we need to search
$$
\begin{align}
0 + 1 + 2 + ... + (N - 1)
&= \frac{ N \cdot (N - 1) }{ 2 } \\
&= \frac{ 1 }{ 2 } \cdot N^2 - \frac{ 1 }{ 2 } N
\end{align}
$$
times to move all the items into $$L_{sorted}$$.
Therefore, the complexity is $$\mathcal{O}(N^2)$$.

## Implementation

See the files on [gist here][gist].

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting.h" data-gist-line="1-14, 17-18, 25"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting.cpp" data-gist-line="1, 5-20, 74-108"></code>

<code data-gist-id="dee9f3bd2ceab69726373ae006016edb" data-gist-file="sorting_test.cpp" data-gist-line="2-37, 45-58, 66-72, 100-101"></code>


The above gist files are imported by [gist-embed][gist-embed].

[sel_sort]: https://chunminchang.gitbooks.io/codeplay/content/sort/selection_sort.html "Selection Sort"
[gist]: https://gist.github.com/ChunMinChang/dee9f3bd2ceab69726373ae006016edb "simple_sort"
[gist-embed]: https://github.com/blairvanderhoof/gist-embed "gist-embed"

<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/gist-embed/2.4/gist-embed.min.js"></script>

[codeplay]: https://www.gitbook.com/book/chunminchang/codeplay/details "CodePlay"
[ins-sort]: https://chunminchang.gitbooks.io/codeplay/content/sort/insertion_sort.html "Selection Sort"
