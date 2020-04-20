---
layout: post
title: How to avoid duplicate symbols when compiling
category: [Programming]
tags: [C/C++]
comments: true
---

# How to avoid duplicate symbols when compiling

The error: ```duplicate symbol for architecture x86_64``` will be prompted
if the following [files][gist] are compiled.

- makefile
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 makefile %}
- main.cpp
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 main.cpp %}
- utils.h
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 utils.h %}
- hello.h
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 hello.h %}
- hello.cpp
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 hello.cpp %}
- bye.h
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 bye.h %}
- bye.cpp
{% gist ChunMinChang/553962a80ac4d873454597d82227c276 bye.cpp %}

You will see the ```duplicate symbol for architecture x86_64``` error
when you compile those files.
```bash
$ make
g++ -c -Wall bye.cpp
g++ -c -Wall hello.cpp
g++ -Wall main.cpp bye.o hello.o -o run
duplicate symbol __Z3LOGNSt3__112basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEE in:
    bye.o
    hello.o
ld: 1 duplicate symbol for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [all] Error 1
```

The reason is that we include the shared header(```utils.h```)
into different files(```bye.cpp``` and ```hello.cpp```),
and compile those files into different libraries(```bye.o``` and ```hello.o```),
so the functions in the shared header(```LOG```)
duplicate in those different libraries.

Thus, when we try using those different libraries(```bye.o``` and ```hello.o```)
at the same time, there are duplicated symbols for functions(```LOG```)
included from the shared header(```utils.h```).
The program has no idea about which one it should call
among those duplicated symbols.

## Approach 1: Using macros instead of functions

The macro is only textual substitution that expanded by the preprocessor,
so there is no symbol generated.

You can replace function ```LOG``` by
```cpp
#define LOG(s) (std::cout << s << std::endl)
```

Then the ```SayBye()``` will be expanded into:
```cpp
void SayBye()
{
  (std::cout << "Goodbye" << std::endl);
}
```

You can run: ```$ g++ -E <file_name>.cpp``` to watch
and confirm the preprocessor's output.

## Approach 2: Make functions __inline__

It works almost same as macro.
Inline functions are actual functions
whose copy of the function body are injected directly into
each place the function is called.

```cpp
inline void LOG(std::string s)
{
  std::cout << s << std::endl;
}
```

The insertion occurs only if
the compiler's cost/benefit analysis shows it to be profitable.
Same as the macros, inline expansion eliminates
the overhead associated with function calls.

Inline functions are parsed by the compiler,
whereas macros are expanded by the preprocessor.
The preprocessor macros are just substitution patterns in code
before the compilation,
so there is no __type-checking__ at that time.
While inline functions are actual functions, so compiler can keep an eye on
type-checking issues to help debugging.

See [here][inline] for more details.

## Approach 3: Using __static__ to make functions local in each file

Since their states are __not__ sharable,
they should __not__ visible across each other.
Thus, the generated symbols are also local in each file.

```cpp
static void LOG(std::string s)
{
  std::cout << s << std::endl;
}
```

[gist]: https://gist.github.com/ChunMinChang/553962a80ac4d873454597d82227c276 "gist"
[inline]: https://chunminchang.gitbooks.io/cplusplus-learning-note/content/Appendix/preprocessor_macros_vs_inline_functions.html#inline-functions "Inline functions"
