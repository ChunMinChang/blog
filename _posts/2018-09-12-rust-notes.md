---
layout: post
title: My Rust Notes
category: [Common]
tags: [Rust]
comments: true
---

# Rust Notes

- Common
  - [Understand borrowing by read-write concepts][borrowed-ptr]
  - How to convert a C strings to Rust strings
  - Phantom data
  - Common Error
    - [Don't misuse the pointers of the instances allocated in functions stack][func-stack]
- Multi-threading
  - [Mutex and RwLock][multithread]
- FFI to C library
  - [A sample Rust library based on platform C APIs][ffi-rust-lib-sample]
  - [Using single-element (tuple) struct to wrap native types][ffi-newtype]
    - [Size of the single-element struct][ffi-newtype-size]
    - [Benefits to wrap the C string][ffi-newtype-cstirng]
    - [An example based on CoreAudio][ffi-newtype-coreaudio]
  - [How to wrap a callback from external library][ffi-callback]
- Taste
  - Error Handling
    - [Error passing from modules to modules][error-passing]
  - Testing
    - Embeding a test module inside every module

[borrowed-ptr]: https://gist.github.com/ChunMinChang/ac1f00e3521755814714436a80d72003 "Learning notes for norrowed pointers"

[multithread]: https://github.com/ChunMinChang/play-multithread "Learning multithread in Rust "

[func-stack]: https://gist.github.com/ChunMinChang/099cd7d88938ad8840dc98e376a8da29 "Don't misuse the pointers of the instances allocated in functions stack"

[ffi-rust-lib-sample]: https://github.com/ChunMinChang/rust-audio-lib-sample/tree/master "rust-audio-lib-sample"
[ffi-newtype]: https://gist.github.com/ChunMinChang/1acf672babd4e8f79fcf83fa228d1461 "Using single-element (tuple) struct to wrap native types"
[ffi-newtype-size]: https://gist.github.com/ChunMinChang/b76a61273374a1530bc4d6f3be6a7761 "Size of the single-element struct"
[ffi-newtype-cstirng]: https://gist.github.com/ChunMinChang/25f3608c285f1abf2a5c289d5f758427 "Using single-element (tuple) struct to wrap C strings"
[ffi-newtype-coreaudio]: https://gist.github.com/ChunMinChang/07b806cb6a9ea1136cb3cbd8cda6c806 "Using single-element (tuple) struct to CoreAudio types"
[ffi-callback]: https://gist.github.com/ChunMinChang/8a22f8a1308b6e0a600e22c4629b2175 "A counterexample to register the callback functions to the external libraries"

[error-passing]: https://gist.github.com/ChunMinChang/92d0006fb9fe35abcabff6983d31f0da "Error passing from modules to modules"
