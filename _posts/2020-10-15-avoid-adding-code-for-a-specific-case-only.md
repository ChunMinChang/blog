---
layout: post
title: Avoid adding code for a specific case only
date: 2020-10-15 05:57 +0800
category: [Programming]
tags: [Software engineering]
comments: true
---
Why is it bad? It branches out a new code path for a specific case and inevitably raises the code complexity. In the long run, its knotty code paths make the code hard to be read and maintained.

This post proposes a way helping the developers to work out a better approach from an approach working only for a specific case.

<!--read more-->

```cpp
void some_function() {
  if (A & B & C & D) {
    // run task1
  }

  if (A & C & E) {
    // run task2
  }

  if (A & B & F) {
    // run task3
  }

  if (A & E) {
    // run task4
  }

  if (P & Q) {
    // run task5
  }

  if (Q & R) {
    // run task6
  }

  // ....
}
```

## Merging divergent code paths

In many cases, the cause leading to problem _X_, in a specific case, usually results to problem _Y_ and _Z_ as well.

That's why we always encourage the programmers to **fix the root cause**, and we always prefer a general approach rather than an approach working only for a specific case. We should try our best to solve these problems at the same time in a common way. However, if we don't know what problem _Y_ and _Z_ are, how do we find out a common way? and how do we know what the root cause is?

### Relax the conditions

If we observe the problem _X_ occurs when `X_1 && X_2 && .. && X_N` is `true`, and we know running function `f` at that time can solve the problem, then relaxing the conditions of when `f` is executed is likely to help us to work out a suitable solution, and help us to find out the root casue. We need to reduce the `N` as much as we can. That helps us to find the **minimal** conditions to trigger/reproduce the problem.

```cpp
void some_function() {
  // ...

  if (X_1 && X_2 && .. && X_N) {
    f();
  }

  // ...
}
```

should be

```cpp
void some_function() {
  // ...

  if (X_1 && X_2) {
    f();
  }

  // ...
}
```

or

```cpp
void some_function() {
  // ...

  f();

  // ...
}
```

In code level, what we want is to reduce the `if (X_1 && X_2 && .. && X_N) { f(); }` to `if (X_1 && X_2) { f(); }` or even removing the `if`(running `f` directly). The problem occurs actually when `true && X_1 && X_2 && .. && X_N` is `true`, so the reducing `if (X_1 && X_2 && .. && X_N) { f(); }` to `if (true) { f(); }` (it's the same as running `f` directly without `if`) sounds reasonable.

In my experience, an intermittent bug happens sporadically and mysteriously can become reproducible when `X_1 && X_2 && .. && X_N` is `true` as well. Running `f` directly, without `if (X_1 && X_2 && .. && X_N)`, to fix that bug makes perfect sense since that bug occurs without those conditions. Those conditions just make the problem become visible. (We are lucky at most of time ;))

On the other hand, relaxing the conditions of when `f` is executed helps us to reduce the unnecessary code branches. In the first example of `some_function`,

```cpp
void some_function() {
  if (A & B & C & D) {
    // run task1
  }

  if (A & C & E) {
    // run task2
  }

  // ....
}
```

if there is no harm in running _task1_ and _task2_ when only `A` is `true` (it's ok to run _task1_ when `B & C & D` is `false` and _task2_ when `C & E` is `false`), we can merge the first-two `if` blocks to one and reduce the code branches:

```cpp
void some_function() {
  if (A) {
    // run task1
    // run task2
  }

  // ....
}
```

By keep doing so, the `some_function` could be:

```cpp
void some_function() {
  if (A) {
    // run task1
    // run task2
    // run task3
    // run task4
  }

  if (Q) {
    // run task5
    // run task6
  }

  // ....
}
```

#### Real-world Example

One example is the process to derive a bug-fix from the [initial bug-fix proposal here](https://bugzilla.mozilla.org/page.cgi?id=splinter.html&ignore=&bug=1646719&attachment=9178827) to a [refined solution here](https://hg.mozilla.org/mozilla-central/rev/c845576c6a34), in [BMO 1646719](https://bugzilla.mozilla.org/show_bug.cgi?id=1646719).

I applied this technique to refine my patch to fix an intermittent bug. I didn't have prior developing experience on the code my patch was added to, but I was able to use this technique to find a suitable solution. That is why I want to share the idea I work out: relax the conditions.

#### Example

Here is another example (inspired by the above one). Suppose we have a big oven (a.k.a, "smart"-oven), and the oven will send a signal notifying us the baking states of its baking stuff. Once the stuff finish baking, we can remove it from the oven. The code(running procedure) within the machine is:

```cpp
void add_to_oven(BakedGood baked_good) {
  queue_good_to_oven(baked_good);
}

void periodical_oven_check() {
  // Bake goods only when power is enough to emit the bake lights
  if (power is enough) {
    if (oven is not full) {
      // Assume the order must be kept
      while (oven-queued goods are not empty) {
        good = first good in the oven queue
        if (good.size > remaining space in oven) {
          break;
        }
        pop the first good
        put the good into oven
        good.state = baking;
        queue_state_update(good);
      }
    }

    for (each good in the oven) {
      temp = check_temperature(good);
      if (temp >= good.target_temp) {
        good.state = finished;
        queue_state_update(good);
        // Users may need to move good out of oven
        // once they get this notification
      }
    }
  }
  // No need to check temperatures if there is no enough power
  // since oven can keep temperature

  // Check if we need to notify the baking states
  while (state-update-queued good is not empty) {
    good = first good in the state-update queue
    pop the first good
    notify_good_state(good);
  }

  schedule the next periodical_oven_check
}
```

It works most of the time. But we notice sometimes we may **never** receive any notification of the backing state for a newly added stuff, when the oven is full or when there is no enough power to bake. (That's an intermittent bug!) The `notify_good_state` is never called for the newly added stuff. To get the signal, we add a function-call to send the baking-state when these conditions are met, once a stuff is put to the waiting-list of the oven.

```cpp
void add_to_oven(BakedGood baked_good) {
  queue_good_to_oven(baked_good);
  if (power is not enough || oven is full) {
    good.state = waiting;
    queue_state_update(good);
  }
}
```

Now we need to ask ourselves: can the following conditions

1. power is not enough
2. oven is full
   
be removed from `add_to_oven`?

The answer is yes. It's no harm to receive an additional state update. In fact, this is what we suppose to do at the first place. It's natural to send a signal reporting the stuff is `waiting` to be baked, once it's queued to be put into the oven.

```cpp
void add_to_oven(BakedGood baked_good) {
  queue_good_to_oven(baked_good);
  good.state = waiting;
  queue_state_update(good);
}
```

The reason why the old procedure works most of the time is because most of the baking stuff is way smaller than the oven size, so the oven is never full, and the power is steady. Hence, most of the time we get the signal reporting the stuff is being `baking` (and it's the first signal we get).

There are other ways to fix this problem. But the above approach is enough to express the idea proposed in this post.

## Closing words

I do hope readers can understand why "adding code for a specific case only" is bad in this post, and the technique proposed, "relaxing the conditions", can really help other developers to work out a better, more general, approach to solve the problem they are working on. Good luck :)