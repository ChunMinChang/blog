---
layout: post
title: Opaque or Transparent Data Type in a Rust Library
category: [Common]
tags: [Rust]
comments: true
---

To develop a *Rust* library that will be used for the external code, the data may be interoperable or non-interoperable. The interoperable data is a data whose underlying memory layout is known and their values can be modified directly by the external code. On the contrary, the non-interoperable data is a data whose underlying memory layout is unknown so their values cannot be changed directly. The underlying values can only be changed when they provide related APIs to do that.

In summary, an interoperable data must be a transparent-type data so it can be modified by external code directly, while a non-interoperable data must be an opaque-type data to prevent its values from being changed by the external code.

In this article, we will explain the above ideas in detail by using some example *Rust* interfaces to *C*.

## Transparent Data Type in Rust
By marking a data type with ```#[repr(C)]```, the data is interoperable with *C/C++* since its memory layout is guaranteed to be aligned with *C*. It can be called as a normal *C* data. For example, you can operate the data with type ```struct Foo``` from *Rust*
```rust
#[repr(C)]
struct Foo {
    bar: i32,
    baz: f32,
    qux: [u32; 5],
}

...

#[no_mangle]
pub extern "C" fn get_foo() -> *mut Foo {
    Box::into_raw(Box::new(Foo::new()))
}

...
```
in *C* code like:
```c
typedef struct {
  int32_t bar;
  float baz;
  uint32_t qux[5];
} Foo;

...

Foo* foo = get_foo();
foo->qux[1] = 100;

...
```

## Opaque Data Type in Rust
For a non-interoperable data, we can just pass a pointer to its memory address with type ```*void```. Without the knowledge of the underlaying data's memory layout, *C/C++* code cannot do anything. 

```rust
// No `#[repr(C)]` here!
struct Foo {
    bar: i32,
    baz: f32,
    qux: [u32; 5],
}

...

#[no_mangle]
pub extern "C" fn get_foo() -> *mut c_void {
    Box::into_raw(Box::new(Foo::new())) as *mut _
}

...
```

```c
...

void* foo = get_foo();
// Cannot do anything with `foo` without knowing the memory layout under this
// void pointer.

...
```
Even the data type is known, it's unwise to cast the ```*void``` to a corresponding *C* data type since the memory layout may be different. That is, the memory layout of the following ```struct Foo``` in *Rust*:
```rust
// No `#[repr(C)]` here!
struct Foo {
    bar: i32,
    baz: f32,
    qux: [u32; 5],
}
```
may be different from the the following ```struct Foo``` in *C*
```c
typedef struct {
  int32_t bar;
  float baz;
  uint32_t qux[5];
} Foo;
```

Therefore, it's dangerous to cast a ```void``` pointer converted by a *Rust*-style ```Foo``` pointer to a *C*-style ```Foo``` pointer to operate it, without marking ```struct Foo``` with ```#[repr(C)]``` in *Rust*.

```c
...

Foo* foo = (Foo*) get_foo();
foo->qux[1] = 100; // This is very likely to fail!

...
```

### Why we need to use Opaque Data Type

The reason why we need to use a *opaque data type* is well described in [Opaque pointer][opa-ptr] and [Opaque data type][opa-data-type] on *Wiki* page. In brief, it gives us the flexibility to change the underlying implementation without changing the interface or the need to recompile the code using the hidden data. 

For example, the following *C* code

```c
...

void* res = get_resource();

...

set_something_to_resource(res, ...)

...
```

based on the following *Rust* code

```rust
struct Foo {
    bar: i32,
    baz: f32,
    qux: [u32; 5],
}

impl Foo {
  fn new() -> Self {
      ...
  }
}

trait SetSomething {
    fn set_something(...);
}

impl SetSomething for Foo {
    fn set_something(...) {
        ...
    }
}

...

#[no_mangle]
pub extern "C" fn get_resource() -> *mut c_void {
    Box::into_raw(Box::new(Foo::new())) as *mut _
}

#[no_mangle]
pub extern "C" fn set_something_to_resource(ptr: *mut c_void, ...) {
    let foo = unsafe { &mut *(ptr as *mut Foo) };
    foo.set_something(...);
    ...
}

...
```

doesn't need to be modified if we change the *Rust* code into
```rust
struct Bar {
    foo: u32,
    baz: f64,
    qux: [i32; 10],
}

impl Bar {
  fn new() -> Self {
      ...
  }
}


trait SetSomething {
    fn set_something(...);
}

impl SetSomething for Bar {
    fn set_something(...) {
        ...
    }
}

...

#[no_mangle]
pub extern "C" fn get_resource() -> *mut c_void {
    Box::into_raw(Box::new(Bar::new())) as *mut _
}

#[no_mangle]
pub extern "C" fn set_something_to_resource(ptr: *mut c_void, ...) {
    let bar = unsafe { &mut *(ptr as *mut Bar) };
    bar.set_something(...);
    ...
}

...
```

[opa-data-type]: https://en.wikipedia.org/wiki/Opaque_data_type "Opaque data type"
[opa-ptr]: https://en.wikipedia.org/wiki/Opaque_pointer "Opaque pointer"