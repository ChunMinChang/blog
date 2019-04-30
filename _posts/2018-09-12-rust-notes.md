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
    - [Panic while panicking][panic_while_panicking]
- Multi-threading
  - [Rules for using mutex][mutex_rule]
  - [The pointers or struct containing pointers cannot be passed across threads][ptr_across_thread]
  - [Mutex and RwLock][multithread]
- Async
  - [oneshot](https://play.rust-lang.org/?gist=e1a1b98654c3490e81d6ff9c262824a3&version=nightly&mode=debug&edition=2018)
- FFI to C library
  - [A sample Rust library based on platform C APIs][ffi-rust-lib-sample]
  - [Opaque or Transparent Data Type][ffi-opa-or-tra-data]
  - [Pass arrays from Rust to C][ffi-rs-array-2-c]
    - [live demo][rs-array-2-c]
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
[mutex_rule]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=use%20std%3A%3Async%3A%3AMutex%3B%0A%0Astruct%20S%20%7B%0A%20%20%20%20m%3A%20Mutex%3Ci32%3E%2C%0A%20%20%20%20x%3A%20u32%2C%0A%7D%0A%0Aimpl%20S%20%7B%0A%20%20%20%20fn%20new()%20-%3E%20Self%20%7B%0A%20%20%20%20%20%20%20%20Self%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20m%3A%20Mutex%3A%3Anew(0)%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20x%3A%200%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20fn%20immutable_borrow(%26self)%20-%3E%20u32%20%7B%0A%20%20%20%20%20%20%20%20self.x%0A%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20fn%20mutable_borrow(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20self.x%20%2B%3D%201%3B%0A%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20fn%20critical_section(%26self)%20%7B%0A%20%20%20%20%20%20%20%20%2F%2F%20Enter%20critical%20section%0A%20%20%20%20%20%20%20%20let%20mut%20guard%20%3D%20self.m.lock().unwrap()%3B%0A%20%20%20%20%20%20%20%20*guard%20%2B%3D%201%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20Leave%20critical%20section%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20s%20%3D%20S%3A%3Anew()%3B%0A%20%20%20%20%0A%20%20%20%20%2F%2F%20Enter%20critical%20section%2C%20borrow%20%60s%60%20immutably%0A%20%20%20%20let%20_guard%20%3D%20s.m.lock()%3B%0A%20%20%20%20%0A%20%20%20%20%2F%2F%20It%27s%20ok%20to%20borrow%20%60s%60%20immutably%20again.%0A%20%20%20%20let%20_%20%3D%20s.immutable_borrow()%3B%0A%20%20%20%20%0A%20%20%20%20%2F%2F%20%60s%60%20cannot%20be%20borrowed%20mutably%20when%20it%27s%20already%20borrowed%20immutably%0A%20%20%20%20%2F%2F%20s.mutable_borrow()%3B%0A%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20belong%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%2B-------------------%2B%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%7C%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7C%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20v%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7C%0A%20%20%20%20%2F%2F%20current%20thread%20%20%20%20%20%20%20%20%20%20mutex%20m%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%7C%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5E%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%7C%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7C%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%2B------------------%2B%0A%20%20%20%20%2F%2F%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20require%0A%20%20%20%20%2F%2F%0A%20%20%20%20%2F%2F%20Lead%20to%20a%20deadlock%20when%20requiring%20a%20locked%20mutex.%0A%20%20%20%20%2F%2F%20s.critical_section()%3B%0A%7D
[ptr_across_thread]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2015&code=%2F%2F%20use%20std%3A%3Aptr%3B%0Ause%20std%3A%3Async%3A%3Ampsc%3A%3Achannel%3B%0Ause%20std%3A%3Async%3A%3A%7BArc%2C%20Mutex%7D%3B%0Ause%20std%3A%3Athread%3B%0A%0A%2F%2F%20If%20the%20struct%20containing%20any%20pointer%2C%20it%20could%20not%20be%20passed%20across%20threads!%0A%23%5Bderive(Debug)%5D%0Astruct%20Data%20%7B%0A%20%20%20%20value%3A%20usize%2C%0A%20%20%20%20%2F%2F%20ptr%3A%20*const%20()%2C%0A%7D%0A%0Aimpl%20Data%20%7B%0A%20%20%20%20fn%20new(value%3A%20usize)%20-%3E%20Self%20%7B%0A%20%20%20%20%20%20%20%20Self%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20value%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%20ptr%3A%20ptr%3A%3Anull()%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20const%20N%3A%20usize%20%3D%2010%3B%0A%0A%20%20%20%20let%20data%20%3D%20Arc%3A%3Anew(Mutex%3A%3Anew(Data%3A%3Anew(0)))%3B%0A%0A%20%20%20%20let%20(tx%2C%20rx)%20%3D%20channel()%3B%0A%20%20%20%20for%20_%20in%200..N%20%7B%0A%20%20%20%20%20%20%20%20let%20(data%2C%20tx)%20%3D%20(Arc%3A%3Aclone(%26data)%2C%20tx.clone())%3B%0A%20%20%20%20%20%20%20%20thread%3A%3Aspawn(move%20%7C%7C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20let%20mut%20data%20%3D%20data.lock().unwrap()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20(*data).value%20%2B%3D%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20(*data).value%20%3D%3D%20N%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20tx.send(()).unwrap()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20rx.recv().unwrap()%3B%0A%20%20%20%20%2F%2F%20data%20may%20still%20be%20locked%20when%20rx%20receiveds%20response%20from%20tx.%0A%20%20%20%20let%20data%20%3D%20data.lock().unwrap()%3B%0A%20%20%20%20println!(%22data%3A%20%7B%3A%3F%7D%22%2C%20*data)%3B%0A%7D%0A
[panic_while_panicking]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=use%20std%3A%3Async%3A%3AMutex%3B%0A%0Astruct%20S%20%7B%0A%20%20%20%20mutex%3A%20Mutex%3Cu32%3E%2C%0A%7D%0A%0Aimpl%20S%20%7B%0A%20%20%20%20fn%20new()%20-%3E%20Self%20%7B%0A%20%20%20%20%20%20%20%20Self%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20mutex%3A%20Mutex%3A%3Anew(0u32)%2C%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20fn%20panic_while_locking(%26self)%20%7B%0A%20%20%20%20%20%20%20%20let%20_guard%20%3D%20self.mutex.lock().unwrap()%3B%0A%20%20%20%20%20%20%20%20panic!()%3B%0A%20%20%20%20%7D%0A%7D%0A%0Aimpl%20Drop%20for%20S%20%7B%0A%20%20%20%20fn%20drop(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20let%20_guard%20%3D%20self.mutex.lock().unwrap()%3B%0A%20%20%20%20%7D%0A%7D%0A%0A%2F%2F%20The%20backtrace%20cannot%20be%20logged%20when%20test%20thread%20panicked%20again%20while%0A%2F%2F%20panicking.%20See%20the%20backtrace%20by%20running%20main.%0A%23%5Btest%5D%0A%23%5Bshould_panic%5D%0Afn%20test()%20%7B%0A%20%20%20%20let%20s%20%3D%20S%3A%3Anew()%3B%0A%20%20%20%20s.panic_while_locking()%3B%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20s%20%3D%20S%3A%3Anew()%3B%0A%20%20%20%20s.panic_while_locking()%3B%0A%20%20%20%20%2F%2F%20After%20panic_while_locking%20is%20called%2C%20the%20s.mutex%20is%20locked%20while%20panicking.%0A%20%20%20%20%2F%2F%20When%20s.drop()%20is%20called%2C%20we%20will%20get%20another%20panic%20when%20requiring%20lock%0A%20%20%20%20%2F%2F%20for%20the%20locked%20s.mutex.%0A%7D%0A

[ffi-rust-lib-sample]: https://github.com/ChunMinChang/rust-audio-lib-sample/tree/master "rust-audio-lib-sample"
[ffi-opa-or-tra-data]: opaque-or-transparent-data-type-in-a-rust-library.md "Opaque or Transparent Data Type in a Rust Library"
[ffi-rs-array-2-c]: https://gist.github.com/ChunMinChang/1e5410f3a7cb8c5bbf066e7dae09d7bc "Pass arrays from Rust to C "
[rs-array-2-c]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6d6c2271e3811d55f740b20a00975ecf "Leak a vec and then retake it"
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
