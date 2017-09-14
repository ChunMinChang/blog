---
layout: post
title: The Misuse of RefPtr
category: Common
tags: [C/C++, Smart Pointer, Firefox]
comments: true
---
In <!--[last post]({% post_url 2017-07-31-refptr-v-s-shared-ptr %})-->
[previous post](https://chunminchang.github.io/blog/post/refptr-v-s-shared-ptr),
I introduced the ```RefPtr<T>``` that can keep tracking the references to the
object ```T``` and the references are counted by object ```T``` itself.

Today, I will note my misuse of it several weeks ago.
This is also why I want to write the posts about ```RefPtr<T>```.

## What behavior I want
The following behavior is what I want when I was implementing one patch
for Firefox:

```c++
List<RefPtr<Solder>> l; // A list containing all of the solders.
{
  RefPtr<Solder> s(new Solder(99)); // Put the `s` into the list.
}
// The `s` is destroyed, so it should be removed from the list now.
```

and we will put the ```Solder``` instance into the ```l``` when
it's created and remove it when it's destroyed.

```c++
Solder()
{
  Add this into `l`
}
~Solder()
{
  Remove this from `l`
}
```

## Why it doesn't work
This is a wrong pattern to meet our expectation.
The solders in the list will **only** be destroyed and removed from the list
when the whole program is ended.
The solders are only removed from the list in its deconstructor.
However, whenever the ```RefPtr<Solder> s(new Solder())```
is deconstructed (by ```~RefPtr```) in the main function,
the ```~Solder()``` won't be called
since there must be one another ```RefPtr<Solder> some``` in the list
referencing the solder.
Thus, the ```~Solder()``` is only be called
when the element in the list is decontructed.

## Sample code
{% gist ChunMinChang/e783052c7da8b4bd5678dbc26de84ab1 misuse.cpp %}
