---
layout: post
title: Shape your code as how you shape your body
category: [Work]
tags: [Firefox, Media, Rust]
comments: true
---

The approach to (re)shape the code is the same as how you shape your body.
You can apply what you've learned from workout to writing code.

<!--read more-->

I've been working on *oxidizing* the Firefox's audio backend, named [*Cubeb*][cubeb],
from [*C++*][cubeb-audiounit] to [*Rust*][cubeb-coreaudio-rs] for more than one year.
The new Rust backend has been shipped in *Firefox 74*.

This is the second post for sharing what I’ve worked out.
I am going to introduce how I make the plan to oxidize the Firefox's audio library.
The first [post][summary] summarizes the [achievements][summary] done in this project.
The final [post][tips] briefs some [tips-and-effects][tips].

The series of this *C-to-Rust* oxidizing story is nothing more than a workout-training post
that you've probably read before:
it makes a plan and lists all the tips a trainee needs
and shows a perfect body shape demonstrating
the results for practicing the plan and applying the tips.

## We Need A Mean Coach, and *Rust Compiler* Is the One

Everyone probably knows a few tricks for making the body in a good shape
but not everyone can achieve that.
Even the shape is made, it's hard to keep the shape.

The quickest way to shape the body is hiring a mean coach!
They make sure you've done everything right and push you to do your best.

Human is lazy. We only move on when something pushes us.

When this habit comes to programming,
I admit that sometimes I don't stick on the coding disciplines the code should follow,
as long as the code pass the tests.
That's why I need a code reviewer.

However, code reviewers aren't always there.
The code can become out of shape a bit and a bit if no one constantly watches it.
This phenomenon reflects
the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory) I've read.

To keep the code in a good shape, we need a robot-reviewer that is always accountable and trustworthy.
The **Rust compiler** is definitely one of them.

If you google *Rust*, you will see the comments are largely divided into two groups:
Some complain the *Rust compiler* is too stupid to allow them to do things they want;
Some love the *Rust compiler* since it helps them to avoid doing stupid things.

The same situation can happen when asking the students to comment a strick teacher or professor:
Some love learning things the hard way at the beginning;
Some love learning things roughly and quickly first then try improving.

Gladly I am an Asian. I am totally fine with a mean coach.
I bet I've seen meaner one in my school life.
The *Rust compiler* is the robot-reviewer I need.

## Motive to Have A Good Shape

There are some reasons for supporting the decision
to shape the audio library in *Rust*:

1. This could reduce the developing and debugging efforts in the long run
2. Some obscure problem may be addressed by this deep-cleaning process
3. It's time to refactor the library after putting hot-fix and hot-fix

Before this project, our team have successfully translated
the *PulseAudio* backend on *Linux* from *C* to *Rust* so
we believe the plan is feasible.

By oxidizing all the code into *Rust*, which was born with good coding principals,
the human effor to review or check the code could be reduced.
In addition, the *Rust*'s eco-system and community is thriving and robust.
The library can leverage many useful third-party *Rust crate*s to fulfil our needs.
Using the third-party library can not only reduces the maintaining effort
but also __embody the spirit of *open-source*__.

When translating the *C* code to the *Rust* code that follows those strict *Rust* rules,
some mysterious issues might be addressed silently
when transforming the code into a stricter structure (we pray 🙂 | _update: I've confrimed some crashes stop after Rust backend is on the right track_).

The code would need to be refactored at some point anyway.
*Rust* seems to be the best choice for refactoring the library at this time.

## Define the Desired Shape

The goal is straightforward enough:
translate the library into *Rust* and don't introduce regressions.

In the product-level view, this is pretty similar to:
create a new flight engine by the state-of-the-art techniques,
put it in our current flights, and make our flights soar!

The goal is simple in high-level view, but the detailed process is hard.

After thinking this carefully,
there are some difficulties to make the library into a perfect *Rust* shape:

- It's hard to follow all the *Rust* rules when writing *Rust* itself.
  It would be harder to write code in a certain way we used to do in *C* code
  (especially I didn't have much *Rust* experience at that time)
- It's hard to prevent regressions.
  The library code is appended patch by patch and not every patch comes with a test.
  Without the check to prevent breaking each fix, it's hard to tell
  if one fix for a certain problem would be missing or not
  during translating the code

Instead of worrying about shaping the code body in a certain way
and the side-effects may be introduced,
it's best to make a plan to address or prevent them as best as we can.

The first step is to review the problems we have.

## Review What the Problems Are

> “If I had an hour to solve a problem
> I'd spend 55 minutes thinking about the problem
> and 5 minutes thinking about solutions.”
>
> ― Albert Einstein

I believe how well we define a problem determines how well we solve it.

To bring value of the oxidizing project as much as possible,
the following problems we used to have in the past need to be addressed:

1. Some assertions will be hit unexpectedly
2. Some code are suspected to be executed in a wrong time
3. A fix for one problem may cause another problem or regression
4. We have platform-dependent problem but we only have high-level cross-platform intergretion tests
5. Some issues are device-related and we don't have any way to simulate those device operations

Apparently, the test coverage needs to be enlarged.

1. The _3_ can be addressed if test case can be written more often in an easier way.
2. After separating the *Mac OS*'s platform-dependent code into a standalone *Rust crate*,
the platform-dependent tests can be easily implemented without considering other platforms
so _4_ can be solved.
3. To address _5_, it needs to find a way to simulate the device operations.
4. The _2_ might indicate some data-racing issues.
Our library manages threads inside itself.
The input and output I/O threads will be created when the audio starts playing.
The task threads would be created when the device is switching, plugging, and unplugging.
Writing multi-thread tests is helpful to hunt the potential data-racing issues
and again the APIs to simulate device operations are really necessary.
1. The _1_ may be caused by calling APIs in a wrong time
or those problems only happen on specific hardwares.
By separating a large library API into several smaller APIs,
with propery unit tests and logs, can help us narrow down the scope of the causes.
It's easier to identify a problem by testing several small APIs independently
than testing a large API at once.

## Define the Goals

The requirements can be organized to:

- Enlarge test coverage
  - Find a way to simulate device switching, plugging, and unplugging
  - Create device-related test
  - Create multi-thread test to hunt the potential data-racing issues
  - Create unit tests for each API
- Restructure the library APIs
  - Follow the Rust *rules*
  - Deconstruct a large API into several smaller one
- Solve the issues we hunt during this deep-cleaning

## Make A Feasible Plan

There are two big problems left:

1. How to prevent the regression from being introduced by oxidizing the whole library,
   without having enough tests?
2. I don't have much *Rust* experience

For the problem _1_, the safest approach is: 
translating all the code on a line-by-line basis at first.

All the *C/C++* code should be translated to *Rust* plainly even they break *Rust*'s rule.
*Rust* has `unsafe` block that allows developers to do what they used to do in *C*.
This could move all the workarounds for some special situations or devices
the library used to have at the same time when all the *C* code is rewritten in *Rust*.

One the other hand, the answer of the problem _2_ can be found
on any advertisement flyer in the mailbox:
"We provide a month-free trial class for you to explore and experience!".

Same manner can be applied here.
Writing a minimum-viable audio library in pure *Rust* first as a trial-run
can address problem _2_ effectively. This process also brings many benefits:

- It's able to see what the problems the oxidizing project will face are
  and how to solve them in a way smaller scope
- It shows a rough outline about what the audio APIs should look like at the end,
  since it's rewritten in pure *Rust*
- Some APIs are reusable in the future and they've been tested in a way smaller scope

As a result, the plan is divided into 4 phases:

1. Write a minimum-viable version of the audio library as a trial-run
2. Translate all the lines plainly
   - Add the proper unit tests and intergretion tests in this phase
3. Remove the `unsafe` code introduced in previous stage
   - Now tests are created to support the refactoring
   - Shipping this version in the *Firefox Nightly* to test the new APIs in the wild
4. Keep refactoring the abnormal *Rust* code
   - Leverage with third-party *crate*s to reduce the maintaining efforts
   - Split some code into sub *crate*s to attract some contributors

## Start the Journey

Now, the only things I need is patience, just like what I need when doing workout.
I know everything I should do but the plan is not easy to be stuck with.
Translating code on a line-by-line basis is dull as doing workout set.
Writing test to prevent regression is uninteresting as
calculating *TDEE (Total Daily Energy Expenditure)* to prevent my body from getting fat.

It's is a long, hard, and lonely process.

Not every API is translatable.
Sometimes it's not difficult to find workarounds or alternative approaches.
But sometimes I need to re-implement the whole API that is originally
supported by *Mac*'s compiler by myself.
Most of the APIs involve the memory operations that need to be dealt with carefully.
Memory misusages or corruptions can lead to some serious problem.

Defusing our custom mutex and replacing it by
standard Rust `Mutex<T>`, or task queue, is not an easy job.
The data-racing issues can be easily introduced during this replacing process.

However, all the hard works are worthwhile at the end when the shape is made!
(To be clear, only the shape of code is made.)

## Result Summary

The results is summarized in [this post][summary].

## Tips and Effects

Some tips I find useful are listed in [this post][tips].

[summary]: summary-of-cubeb-oxidation-on-mac-os
[tips]: the-effect-of-practicing-what-you-already-know

[cubeb]: https://github.com/kinetiknz/cubeb
[cubeb-audiounit]: https://github.com/kinetiknz/cubeb/blob/master/src/cubeb_audiounit.cpp
[cubeb-coreaudio-rs]: https://github.com/ChunMinChang/cubeb-coreaudio-rs