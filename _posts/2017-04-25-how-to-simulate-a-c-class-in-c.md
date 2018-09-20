---
layout: post
title: How to simulate a C++ class in C
category: Common
tags: [C/C++]
comments: true
---

# How to simulate a C++ class in C

After learning _C++_, I am curious about
how could I bring the _object-oriented_ style to _C_.
Could we simulate a _C++_ ```class``` by ```struct``` in C?

To find the answer,
I implement a simple _linked list_ with _object-oriented_ style in plain _C_.
The key is to use _function pointer_ in ```struct``` to
simulate a _C++_ member function.
you can find my code [here][gist_c]

To compare, I also implement a [_linked list_ in _C++_][gist_c++].

## Class in C
{% gist ChunMinChang/31f11789bb859356daf05107e8fc859e list.h %}
{% gist ChunMinChang/31f11789bb859356daf05107e8fc859e list.c %}
{% gist ChunMinChang/31f11789bb859356daf05107e8fc859e test.c %}

## C++ version
{% gist ChunMinChang/8e04130e778d77e0b30b8954cc5f2473 list.h %}
{% gist ChunMinChang/8e04130e778d77e0b30b8954cc5f2473 test.cpp %}

## Comparison
- The C version needs to call ```destroy``` explicitly,
  while the C++ version will automatically run deconstructor ```~List()```
  to release the memory, or use smart pointers
  like ```unique_ptr``` to help memory management.
  - To release ```Foo* n = new Foo(...)```, we need to use ```delete n```
    instead of ```n->~Foo()```
    - Calling a destructor releases the resources owned by the object,
      but it does not release the memory allocated to the object itself.
- We need to pass self pointer to the ```List``` structure
  for calling functions to access list's data,
  while we don't need to do that in C++ version
  because class object can get all data inside itself in its implementation.
- To allow storing different data type in the list,
  the C++ version use ```template``` instead of ```void*``` in the C version.
  - The ```void* data``` with ```size_t size```
    is regarded as memory chunk beyond types,
    pointed by ```data``` with ```size``` bytes,
    so we can store different types data in __runtime__.
  - While ```template<typename T>``` let us to declare a variable
    with type ```T``` in __compile time__,
    so _gcc/g++_ can help us for debugging if there is any error.
    - function with ```template``` cannot be separated in ```.cpp``` and ```.h```
      because compiler needs to see both the template definition
      and the specific types/whatever used to __fill in__ the template.
      Please read [this][why] for more details.
- Replace ```NULL``` with ```nullptr```
  - ```nullptr``` is always a pointer type. ```NULL```(0) could cause ambiguity
    when we have functions: ```void f(int)```, ```void f(foo *)```,
    and we call ```f(NULL)```.
    
## References
- [你所不知道的 C 語言：物件導向程式設計篇（共筆）][jserv-ooc]
- [你所不知道的 C 語言：物件導向程式設計篇 video (上)][jserv-video-ooc-A]
- [你所不知道的 C 語言：物件導向程式設計篇 video (下)][jserv-video-ooc-B]

[gist_c]: https://gist.github.com/ChunMinChang/31f11789bb859356daf05107e8fc859e "Class in C for linked-list implementation"
[gist_c++]: https://gist.github.com/ChunMinChang/8e04130e778d77e0b30b8954cc5f2473 "Linked-list in C++"
[why]: https://isocpp.org/wiki/faq/templates#templates-defn-vs-decl "Why can’t I separate the definition of my templates class from its declaration and put it inside a .cpp file"

[jserv-ooc]: http://hackfoldr.org/dykc/https%253A%252F%252Fhackmd.io%252Fs%252FHJLyQaQMl "你所不知道的 C 語言：物件導向程式設計篇（共筆）"
[jserv-video-ooc-A]: https://www.youtube.com/watch?v=GON5vID8srE "你所不知道的 C 語言：物件導向程式設計篇 (上)"
[jserv-video-ooc-B]: https://www.youtube.com/watch?v=k0k9ZK64g1s "你所不知道的 C 語言：物件導向程式設計篇 (下)"
