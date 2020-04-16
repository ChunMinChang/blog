---
layout: post
title: Shape your code as how you shape your body
category: [Media]
tags: [Firefox]
comments: true
---

The approach to refactor the code is same as how you shape the body.
You can apply what you've learned from workout to writing code.

<!--read more-->

This post introduce how I make the plan for rewriting the Firefox's audio library,
named [*Cubeb*][cubeb], from [*C++*][cubeb-audiounit] into [*Rust*][cubeb-coreaudio-rs].

The story is nothing more than a workout-training post that you've probably read before:
it lists all the tips a trainee needs and shows a perfect body shape demonstrating
the results for applying those tips.

Everyone probably knows a few tricks for making the body in a good shape
but not everyone can achieve that.
Even the shape is made, it's hard to keep the shape.

The quickest way to shape the body is hiring a mean coach!
They make sure you've done everything right and push you to do your best.

Human is lazy. We only move forward when something pushes us.

## We Need A Mean Coach, and *Rust Compiler* Is the One

When this habit comes to programming,
I admit that sometimes I don't stick on the coding disciplines the code should follow,
as long as the code pass the tests.
That's why I need a code reviewer.

However, code reviewers aren't always there.
The code can become out of shape a bit and a bit if no one constantly watches it.
This phenomenon reflects
the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory) I've read.

To keep the code in a good style, we need a robot-reviewer that is always accountable and trustworthy.
The **Rust compiler** is definitely one of them.

If you google *Rust*, you will see comments largely divided into two groups:
Some complain the *Rust compiler* is too stupid to allow them doing things they want;
Some love the *Rust compiler* since it makes the code well-structured
and prevent them from doing stupid things.
This is same as the comments for a strict teacher or professor.

Gladly I am an Asian. I am totally fine with a mean coach.
I bet I've seen meaner one in my school life.
The *Rust compiler* is the robot-reviewer I need.

## Motive to Have A Good Shape

There are some reasons for supporting the decision
to shape the audio library in *Rust*:

1. This could reduce the developping and debugging efforts in the long run
2. Some mysterious problem may be addressed by this deeply cleaning process
3. It's time to refactor the library after putting hot-fix and hot-fix

Before this project, our team have successfully translated
the *PulseAudio* backend on *Linux* from *C* to *Rust* so
we believe the plan is feasible.

By rewriting all the code in *Rust*, which was born with good coding principals,
the human effor to review or check the code would be lower.
In addition, the *Rust*'s eco-system and community is thriving and robust.
The library can leverage many useful third-party *crate*s to fulfil our needs.
Using the third-party library can not only reduces the maintaining effort
but also __embody the spirit of *open-source*__.

When translating the *C* code to the *Rust* code that follows those strict *Rust* rules,
(we pray) some mysterious issues might be addressed silently
with transforming the code structure in a stricter style.
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

To bring value of the rewriting project as much as possible,
the following problems need to be addressed:

1. Some assertions will be hit unexpectedly
2. Some code are suspected to be executed in a wrong time
3. A fix for one problem may cause another problem or regression
4. We have platform-dependent problem but we only have high-level cross-platform intergretion tests
5. Some issues are device-related and we don't have any way to simulate those device operations

Apparently, the test coverage needs to be enlarged.

The _3_ can be addressed if test case can be written more often in a easier way.

After separating the *Mac OS*'s platform-dependent code into a standalone crate,
the platform-dependent tests can be easily implemented without considering other platforms
so _4_ can be solved.

To address _5_, it needs to find a way to simulate the device operations.

The _2_ might indicate some data-racing issues.
Our library manages threads inside itself.
The output and input I/O threads will be created when the audio starts playing.
The task threads would be created when the device is switching, plugging, and unplugging.
Writing multi-thread tests is helpful to hunt the potential data-racing issues
and again the APIs to simulate device operations are really necessary.

The _1_ may be caused by calling APIs in a wrong time
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

1. How to prevent the regression from being introduced by rewriting the whole library, without having enough tests?
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

- It's able to see what the problems the rewriting will face are and how to solve them,
  in advance, in a way smaller scope
- It shows a rough outline about what the audio APIs should look like at the end,
  since it's rewritten in pure *Rust*
- Some APIs are reusable in the future and they've been tested in a way smaller scope

As a result, the plan is divided into 4 phases:

1. Write a minimum-viable version of the audio library as a trial-run
2. Translate all the lines plainly
   - Add the proper unit or intergretion tests at the same time
3. Remove the `unsafe` code introduced in previous stage
   - Now tests are created to support the refactoring
   - Shipping this version in the *Firefox Nightly* to test the new APIs in the wild
4. Keeping refactoring the abnormal *Rust* code
   - Leverage with third-party crates to reduce the maintain efforts
   - Split some code into sub crates to attract some contributors

## Start the Journey

Now, the only things I need is patience, just what I need when doing workout.
I know everything I should do but the plan is not easy to stick with.
Translating code on a line-by-line basis is dull as doing workout set.
Writing test to prevent regression is uninteresting as
calculating *TDEE (Total Daily Energy Expenditure)* to prevent my body from getting fat.

It's is a long, hard, and lonely process.
But I find everything worth doing when the shape is finally made!
(To be clear, only the shape of code is made.)
I am happy to have the result I have at the end.

### Results

The result summay is listed in [this post][summary].

### Tips

Some tips I find useful is listed in [this post][tips].

[summary]: summary-of-cubeb-oxidation-on-mac-os
[tips]: the-effect-of-practicing-what-you-already-know

[cubeb]: https://github.com/kinetiknz/cubeb
[cubeb-audiounit]: https://github.com/kinetiknz/cubeb/blob/master/src/cubeb_audiounit.cpp
[cubeb-coreaudio-rs]: https://github.com/ChunMinChang/cubeb-coreaudio-rs