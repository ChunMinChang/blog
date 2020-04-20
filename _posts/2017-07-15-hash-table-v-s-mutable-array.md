---
layout: post
title: Hash Table v.s. Mutable Array
category: [Programming]
tags: [C/C++]
comments: true
---
It's common sense that the time complexity of insertion and removal of a
hash table are all both _O(1)_, while array takes _O(n)_ for removal.
However, when the data size(_n_) is small, the array will beat the hash table.

Here is the result from my test[(gist here)][gist]
{% gist ChunMinChang/b6b7b534e1ef3683f76d830d72c489a6 README.md %}

By the results, if you are pretty sure the data size is less than ```50```
then you should use ```std::vector``` instead of ```std::unordered_map```.

On the other hand, if you need to insert and remove itmes frequently,
and the data size is greater than ```50```,
then you should use ```std::unordered_map``` instead of ```std::vector```.
If items are inserted frequently but removed rarely, ```std::vector``` is fine.

[gist]: https://gist.github.com/ChunMinChang/b6b7b534e1ef3683f76d830d72c489a6 "Performance: Mutable array v.s. Hashtable"
