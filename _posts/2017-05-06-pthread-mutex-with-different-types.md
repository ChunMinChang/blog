---
layout: post
title: Pthread mutex with different types
category: [Programming]
tags: [Multithread]
comments: true
---
When I tried to [produce a deadlock in _CoreAudio_][DL-AU] with pthread,
I realized that the _mutex_ with __NORMAL__ type locked by one pthread
could be unlocked by another pthread.
Normally, this behavior should be __disallowed__.
It will result in undefined behaviors.
If one pthread could unlock a mutex owned by other thread whenever it wants,
then the mutex will be meaningless,
unless it's a expected behavior.

In my case, it's exactly what I want,
because I need to break the deadlock to continue the program.
However, in most case, this behavior should be __forbidden__,
so I do some research about
what the behaviors of mutexes with different types are.

Here is my conclusion:

{% gist ChunMinChang/fe46f3760e2230c09675e258ae1cf8eb README.md %}

The test code can be found on [gist here][gist]

[DL-AU]: deadlock-when-using-audiounit
[gist]: https://gist.github.com/ChunMinChang/fe46f3760e2230c09675e258ae1cf8eb
