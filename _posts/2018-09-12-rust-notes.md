---
layout: post
title: My Rust Notes
category: [Common]
tags: [Rust]
comments: true
---

My personal Rust notes.

<!--read more-->

# Rust Notes

## Learning Resources
- Quick guide for system programmer
  - [30 minutes of Introduction to Rust for C++ programmers](https://legacy.gitbook.com/book/vnduongthanhtung/migrate-from-c-to-rust/details)
  - [Rust For Systems Programmers](https://github.com/nrc/r4cppp)
- Official books
  - [The Rust Programming Language](https://doc.rust-lang.org/book/)
  - [The Rust Programming Language(book)](https://nostarch.com/Rust)
  - [Programming Rust](http://shop.oreilly.com/product/0636920040385.do)

## Personal Notes
- Common
  - [Understand borrowing by read-write concepts][borrowed-ptr]
  - [Lifetime][lifetime]
  - How to convert a C strings to Rust strings
  - Phantom data
  - Common Error
    - [Don't misuse the pointers of the instances allocated in functions stack][func-stack]
    - [Temporary cstring as ptr][temporary_cstring_as_ptr]
- Multi-threading
  - [Mutex and RwLock][multithread]
- Async
  - [oneshot](https://play.rust-lang.org/?gist=e1a1b98654c3490e81d6ff9c262824a3&version=nightly&mode=debug&edition=2018)
- FFI to C library
  - [A sample Rust library based on platform C APIs][ffi-rust-lib-sample]
  - [Opaque or Transparent Data Type][ffi-opa-or-tra-data]
  - [Pass arrays from Rust to C][ffi-rs-array-2-c]
  - [The size matters! How to mess up memory by using vector in a wrong way][ffi-vec-size]
  - [A counterexample to use the memory allocated in external library][ffi-memory]
  - [Using reference /pointer instead of copying to get a variable-sized struct object][ffi-get-variable-sized-struct]
  - [Using single-element (tuple) struct to wrap native types][ffi-newtype]
    - [Size of the single-element struct][ffi-newtype-size]
    - [Benefits to wrap the C string][ffi-newtype-cstirng]
  - [How to wrap a callback from external library][ffi-callback]
  - C, C++, Rust Examples to call C Query APIs
    - [Basic: Wrap native type by tuple struct ][ffi-device-basic]
    - [With String Handle][ffi-device-string]
  - OSX
    - [Single-element tuple structs wrapping native CoreAudio types][ffi-osx-newtype-coreaudio]
    - [Rust wrappers for OSX dispatch APIs][ffi-osx-dispatch]
    - [Rust wrappers for OSX property listner on audio devices][ffi-osx-audio-property-listener]
    - Rust wrappers for OSX CFString(Ref) APIs
- Taste
  - Error Handling
    - [Error passing from modules to modules][error-passing]
  - Testing
    - Embeding a test module inside every module

[borrowed-ptr]: https://gist.github.com/ChunMinChang/ac1f00e3521755814714436a80d72003 "Learning notes for norrowed pointers"

[lifetime]: https://gist.github.com/ChunMinChang/e8096bc78d29b237cce3ff5f859834e7 "Lifetimes for The Rust References"

[multithread]: https://github.com/ChunMinChang/play-multithread "Learning multithread in Rust "

[func-stack]: https://gist.github.com/ChunMinChang/099cd7d88938ad8840dc98e376a8da29 "Don't misuse the pointers of the instances allocated in functions stack"
[temporary_cstring_as_ptr]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=dda40d0b40a8d922649521544f260a91 "temporary cstring as ptr"

[ffi-rust-lib-sample]: https://github.com/ChunMinChang/rust-audio-lib-sample/tree/master "rust-audio-lib-sample"
[ffi-opa-or-tra-data]: opaque-or-transparent-data-type-in-a-rust-library.md "Opaque or Transparent Data Type in a Rust Library"
[ffi-rs-array-2-c]: https://gist.github.com/ChunMinChang/1e5410f3a7cb8c5bbf066e7dae09d7bc "Pass arrays from Rust to C "
[ffi-vec-size]: https://gist.github.com/ChunMinChang/27c7edb4ec45d61a1e8a788888f665cb "A mistake when using a Rust vector as a buffer to get the data by a C API"
[ffi-memory]: https://gist.github.com/ChunMinChang/3f380eaced6265ab6e8dbb224bfec732 "A counterexample to use the memory allocated in external library"
[ffi-get-variable-sized-struct]: https://gist.github.com/ChunMinChang/e8909506cfca774f623fc375fc8ee1d2 "Using reference /pointer instead of copying to get a variable-sized struct object"
[ffi-newtype]: https://gist.github.com/ChunMinChang/1acf672babd4e8f79fcf83fa228d1461 "Using single-element (tuple) struct to wrap native types"
[ffi-newtype-size]: https://gist.github.com/ChunMinChang/b76a61273374a1530bc4d6f3be6a7761 "Size of the single-element struct"
[ffi-newtype-cstirng]: https://gist.github.com/ChunMinChang/25f3608c285f1abf2a5c289d5f758427 "Using single-element (tuple) struct to wrap C strings"
[ffi-osx-newtype-coreaudio]: https://gist.github.com/ChunMinChang/07b806cb6a9ea1136cb3cbd8cda6c806 "Using single-element (tuple) struct to CoreAudio types"
[ffi-callback]: https://gist.github.com/ChunMinChang/8a22f8a1308b6e0a600e22c4629b2175 "A counterexample to register the callback functions to the external libraries"
[ffi-device-basic]: https://gist.github.com/ChunMinChang/1acf672babd4e8f79fcf83fa228d1461 "Using single-element (tuple) struct to wrap native types"
[ffi-device-string]: https://gist.github.com/ChunMinChang/22a30f214c97609d72f17d80740b8506 "C, C++, Rust Examples to call C-compatible Query APIs"


[ffi-osx-dispatch]: https://gist.github.com/ChunMinChang/8d13946ebc6c95b2622466c89a0c9bcc "Rust wrappers for OSX dispatch apis"
[ffi-osx-audio-property-listener]: https://gist.github.com/ChunMinChang/f0f4a71f78d1e1c6390493ab1c9d10d3 "Rust wrappers for OSX property listner on audio devices"
[error-passing]: https://gist.github.com/ChunMinChang/92d0006fb9fe35abcabff6983d31f0da "Error passing from modules to modules"
