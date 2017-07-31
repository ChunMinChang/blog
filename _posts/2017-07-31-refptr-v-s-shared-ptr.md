---
layout: post
title: RefPtr v.s. shared_ptr
category: Common
tags: [C/C++, Firefox]
comments: true
---
When I tried to replace Mozilla's [RefPtr][moz_refptr] with standard C++
smart-pointer to note one of my misusage of it,
I used *std::shard_ptr* to do it.
However, it didn't work since there is a huge difference between
Mozilla's [RefPtr][moz_refptr] and the *std::shard_ptr*

I've never noticed that because I never use smart-pointer
before I worked for Mozilla.
So it's a perfect chance for me to get closer to look at their difference.

## Reference count
Although they're both using reference-count to track object
and manage objects' life-time by the count,
they are counting on different things.

Suppose we have reference-counted pointers ```RefPtr<T>```
and ```std::shard_ptr<T>```,
we will call it like ```RefPtr<Foo> p(new Foo(...))```
and ```std::shard_ptr<Bar> q(new Bar(...))```

### *RefPtr*
When using ```RefPtr<T>```,
the total reference is counted on the ```Foo``` objects.
That's see an example below.
```c++
// Foo must provide reference-counted interface.
Foo* f = new Foo(...) // The total references will be counted on Foo object f.
RefPtr<Foo> p1(f) // the reference count of f is 1 now.
{
  RefPtr<Foo> p2(f) // the reference count of f is 2 now.  
}
// the reference count of f is back to 1 now since p2 is destroyed.
```

### *std::shard_ptr*
When using ```std::shard_ptr<T>```,
the total reference is counted on the ```std::shard_ptr``` itself.
That's see an example below.
```c++
// Bar does NOT need to provide reference-counted interface.
Bar* b = new Bar(...)
std::shard_ptr<Bar> p1(f) // the reference count of p1 is 1 now.
{
  std::shard_ptr<Bar> p2(p1) // the reference count of p1 and p2 is 2 now.
}
// the reference count of p1 is back to 1 now since p2 is destroyed.

std::shard_ptr<Bar> p3(b) // the reference count of p3 is 1 now.
```
Even worse, the above program will cause an error:
**pointer being freed was not allocated**.
The ```p1``` and ```p3``` both control the life-time of ```b```.
When ```p1``` is destroyed, its reference-count is down to ```0```,
so it will deallocate ```b```.
Nevertheless, When ```p3``` is destroyed,
its reference-count is also down to ```0```,
so it will deallocate ```b``` **again** and cause an error:
**Freeing an already-freed object**

Thus, using
```c++
std::shard_ptr<Bar> p(new Bar(...))
```
to replace the following pattern should save your life.
```c++
// This is a bad pattern!
Bar* b = new Bar(...)
std::shard_ptr<Bar> p(f)
```

## Sample code
That's see the example-implementation of these two smart-pointers.
Again, the key difference between *RefPtr* and *std::shard_ptr* is
- ```RefPtr<T> p(new T(...))```:
the reference is counted on the newed ```T``` object
so the ```class T``` must provide *reference-counted* interface.
- ```std::shard_ptr<Bar> q(new Bar(...))```:
the reference is counted on the ```q``` itself.

{% gist ChunMinChang/e783052c7da8b4bd5678dbc26de84ab1 RefPtr.h %}
The above code is referenced from [here](http://www.aristeia.com/BookErrata/M29Source.html)

[moz_refptr]: http://searchfox.org/mozilla-central/source/mfbt/RefPtr.h "RefPtr"
