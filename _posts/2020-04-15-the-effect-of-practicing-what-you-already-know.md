---
layout: post
title: The effect of practicing what you already know
category: [Media]
tags: [Firefox]
comments: true
---

This post presents the results of applying a few tricks every *Rust* developer may already know
but haven't got a chance to try them.

<!--read more-->

I am going to share some actions and results I find useful when
rewriting the Firefox's audio library, named [*Cubeb*][cubeb],
from [*C++*][cubeb-audiounit] into [*Rust*][cubeb-coreaudio-rs].

## What the problem we have

The problem needs to be solved in the rewriting project can be found in [the post here][plan].

In short, the following problems motivate us:

- The library code becomes less-structured after putting hot-fix and hot-fix
- A fix for one problem may cause another problem or regression
- We have platform-dependent problem but we only have high-level cross-platform intergretion tests
- Some issues are device-related and we don't have any way to simulate those device operations
- There must be some data racing issues but the causes are not easy to be identified

The goals are set as follow:

- Enlarge test coverage
  - Find a way to simulate device switching, plugging, and unplugging
  - Create device-related test
  - Create multi-thread test to hunt the potential data-racing issues
  - Create unit tests for each API
- Deconstruct a large API into several smaller one with error messages
- Solve the data-racing, memory leaks, or any issues we hunt during this deep-cleaning

## Summary of the Results

The results we have can be found in [the post here][summary].

In short, the following results are made:

- Solve *6* data racing issues discovered by enlarging the test coverage
  - Some issues exist for ages but their causes are not easy to be identified
- Boost the performance to *35x* faster when starting multiple streams simultaneously
  - A happy side effect when fixing data racing issues
- Hunt and fix *3* memory leaks
- The test coverage is enlarged to almost 80%
  - The left 20% are mostly logs
  - The amount of the regression bugs is pretty low:
    - For now, only *5* bugs are introduced by using the new *Rust* API

## The Effect of Practicing What You Already Know

Everyone probably knows a few tricks for making the body in a good shape
but not everyone can achieve that.
Sticking with the appropriate plan is the key to meet the goals for having a good shape.

I find I can apply what I've learned from workout to writing code.
The approach to refactor the code is same as how we shape the body.

It's is a long and lonely process and the plan is not easy to stick with.
But I find everything worth doing when the shape of code is finally made!

The followings introduce some useful tips I learned from this rewriting project.

To read more story of how the rewriting plan is made,
please read [the post here][plan].

### Always Write Tests

Every programmer knows it's better to write tests
but not every allocates time to do that.

The experience I learned from this rewriting project makes me
believe that the test cases are the founding blocks to
build the above achievements.

`cargo test` run tests in parallel by default.
As a result, some data-racing issues could be naturally detected.

Writing tests can also provide a different view to review the API.
In my case, the idea about how to implement APIs to simulate the device plugging and unplugging
was inspired when running these new added tests.

There is an API used in the library creates an hidden device silently
and fire the device-changed operations.
Therefore that API can be reused to simulate the device plugging and unplugging.
I didn't get this idea before
until I find this API interferes the tests for device-changed operations.

We didn't have any way to simulate the device operations.
Now we have one!

### Turn on Sanitizers

Probably every developer knows the [sanitizers][sanitizers] is useful for system programming.

Enabling the *sanitizer* in a large project is not an easy task
since it needs to tune the compiler settings correctly.

Gladly, *sanitizer* can be enabled easily in *Rust* by `RUSTFLAGS="-Z sanitizer=<SAN_NAME>"` flag.

`RUSTFLAGS="-Z sanitizer=thread" cargo test` helps us to hunt one data-racing issue
and `RUSTFLAGS="-Z sanitizer=leak" cargo test` helps us to fix one memory leak.

It's better to enable *sanitizer* when the Rust *crate* is small
so the compiler setting error for *sanitizer*
introduced by the (newly added) dependent crates can be found earlier.

### Run *XCode*'s *Instruments* with the Test Executable

The test executable generated by `cargo test` can be loaded to *XCode*'s *Instruments* easily.

Running the test executable with *Leaks* check help us to hunt two memory leaks.

### Monitor the test coverage

The [grcov][grcov] is a convenient tool that help monitoring the test coverage in our code.
It can show the test-coverage status of the *Rust* project in just a few line sciprts.

It's useful to know which part has not been tested
since it indicates where we should focus next.

### Use Benchmark to Evaluate the Tradeoff

One problem usually can be solved in many different ways.
Sometime it's hard to tell which approach is most appropriate for the needs.

When performance is one of key factor to make the decision,
`cargo bench` can provide the data to evaluate the approachs.

In this project, `cargo bench` helps us to recognize
one improvement is *4x* faster than the original code.
It also helps us to choose a slightly slower approach with better code readability
over a slightly faster approach with poor code readability
since the perofrmance difference is confirmed to be negligible.

### Ask help when you need

Asking help may be the important facotr I find.

This [post][perfect-team] reveals a fact:
The team will be more productive if the team provide enough *psychological safety*
that makes teammate feel comfortable to ask things freely.

Fortunately, the coworkers I encountered are super supportive.

In my experience, the outcome from a discussion
that starts with an immature idea
is usually better or at least equivalent
than an idea formed by someone alone.
And the process is usually shorter.

### Concolusion

Trying hard to stick with the appropriate plan is the key to shape the code.

It's not necessary to do the plan *100%* correctely.
If you have experience on calculating *TDEE (Total Daily Energy Expenditure)*,
you probably know the goal of fat loss is still reachable
if the achievement rate is only *80~90%*.

Good luck!

[plan]: shape-your-code-as-how-you-shape-your-body
[summary]: summary-of-cubeb-oxidation-on-mac-os

[cubeb]: https://github.com/kinetiknz/cubeb
[cubeb-audiounit]: https://github.com/kinetiknz/cubeb/blob/master/src/cubeb_audiounit.cpp
[cubeb-coreaudio-rs]: https://github.com/ChunMinChang/cubeb-coreaudio-

[sanitizers]: https://github.com/google/sanitizers

[grcov]: https://github.com/mozilla/grcov

[perfect-team]: https://www.nytimes.com/2016/02/28/magazine/what-google-learned-from-its-quest-to-build-the-perfect-team.html