---
layout: post
title: Deadlock when using AudioUnit
category: Media
tags: [CoreAudio, Multithread, Firefox]
comments: true
---
There was a deadlock occured when we tried to integrate my implementation
for audio 5.1 into Firefox.
You can see the [bug here](https://bugzilla.mozilla.org/show_bug.cgi?id=1337805).
It __only happens on OSX__.
After [analysis](https://bugzilla.mozilla.org/show_bug.cgi?id=1350511#c1),
I wrote a test to prevent others from getting into the same problem.
The test is added to [cubeb](https://github.com/ChunMinChang/cubeb),
which is our cross-platform audio library for Firefox.
We reproduced a simpler version of [the deadlock](https://github.com/ChunMinChang/cubeb/blob/8939c0d168a27b1d5047779caad46835ca4651b9/test/test_deadlock.cpp#L1-L43))
in the test.

However, the code is not easy enough for those who are not familir with _cubeb_,
so I wrote a general version to highlight the issue to
everyone who uses _AudioUnit_ in their audio backend.
You can find the code on [gist here](https://gist.github.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9)

{% gist ChunMinChang/47b8712ed57b96721eec18dede39d2f9 test_deadlock.cpp %}

The APIs called to play and stop the audio stream is:
{% gist ChunMinChang/47b8712ed57b96721eec18dede39d2f9 AudioStream.h %}
{% gist ChunMinChang/47b8712ed57b96721eec18dede39d2f9 AudioStream.cpp %}

The key why deadlock happend is that
the audio callback thread holds a mutex(hereafter referred to as _Mutex-AU_)
shared with _AudioUnit_.
The _Mutex-AU_ is held inside it's framework, so you don't notice it.

Thus, if the callback thread requests another _mutex M_ held by the another
thread, without releasing _mutex-AU_, then it will cause a deadlock when the
another thread, which holds the _mutex M_, request to use AudioUnit.


![](https://gist.githubusercontent.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9/raw/e27ea5d4fcee8cfd58acbdea09e90a40a4cfe5e1/deadlock.gif)


That is,
if we have a _thread T_, holding the _mutex M_

![](https://gist.githubusercontent.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9/raw/e27ea5d4fcee8cfd58acbdea09e90a40a4cfe5e1/deadlock-1.png)

and one _callback thread_ which holds the _mutex-AU_,

![](https://gist.githubusercontent.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9/raw/e27ea5d4fcee8cfd58acbdea09e90a40a4cfe5e1/deadlock-2.png)

The deadlock will occur when the _callback thread_ requests the _mutex M_
(the callback thread is blocked for waiting the _mutex M_)

![](https://gist.githubusercontent.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9/raw/e27ea5d4fcee8cfd58acbdea09e90a40a4cfe5e1/deadlock-3.png)

and the _thread T_ requests the _mutex-AU_ to use AudioUnit

![](https://gist.githubusercontent.com/ChunMinChang/47b8712ed57b96721eec18dede39d2f9/raw/e27ea5d4fcee8cfd58acbdea09e90a40a4cfe5e1/deadlock-4.png)