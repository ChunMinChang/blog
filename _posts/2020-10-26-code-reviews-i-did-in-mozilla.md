---
layout: post
title: Code-reviews I did in Mozilla
date: 2020-10-26 01:53 +0800
category: [Work]
tags: [Firefox]
comments: true
---

Giving code-review is an essential part of my daily job as a software engineer. I always try my best to help my peers to address the problem properly when doing a review. For anyone who is interested in my reviews, this post lists some of the code-reviews I did in Mozilla.

<!--read more-->

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Self-evaluation](#self-evaluation)
  - [Comment Style](#comment-style)
  - [Mindset](#mindset)
  - [Review Focus](#review-focus)
- [Code Reviews in Mozilla](#code-reviews-in-mozilla)
  - [Components](#components)
  - [List](#list)
- [Other Reviews and Communications](#other-reviews-and-communications)
- [Closing Words](#closing-words)

**TL;DR**: See my review skills from the [list](#list)

This post organizes some code reviews I did in Mozilla in the past.
I hope this post can be a reference for myself to check if my review skills improve or not when I look back several years later. Aside from that, this is a reference for anyone who is interested in my review quality.

## Self-evaluation

I can review code effectively and help the reviewees to notice the defects in their patches or the potential bugs they miss. There are some examples as follow. What’s more, I am able to explain some subtle software-engineering principles clearly with verbal comments, such as [what is over-engineering and why is it bad][overengineering], or [why is adding code for a specific case bad][specific-code-is-bad]. I am able to deliver clear messages to make my teammates understand what I am talking about exactly so we can make the code better together.

### Comment Style

Tone matters. Different tone makes different impression, yet there is some tones I avoid.

I rarely use "you" in my comments, since what I judge is "code" instead of "author".
I use "why" more often than "it’s wrong". *Wrong* implies I know what exactly *right* is, which is not the case most of the time. *Wrong* means the *current result* is different from the *expected result*. But the *expected result* is often vague. In that case, there is no *wrong* and *right*. I always compare the *pros* and *cons* between the different approaches, instead of saying *wrong* and *right* based on my personal judgements. I only use *wrong* when the *expected result* is clearly *define*d (e.g., by a certain spec). Asking *Why* is a better way to know the reason for making a certain trade-off.

### Mindset

Be a collaborator instead of an authority.

I don’t need to "guide" the patch author to do their jobs in a way I prefer. For me, this is one kind of [*microaggression*](https://en.wikipedia.org/wiki/Microaggression). I don’t have *right* to decide what they should do. The word *guide* kind of means leading the code auther to do what I expect, but which subtly implies or preassume I know better what to do than them. I know sometimes the review for a patch asking for changing code in *X* field will be requested by the contributor without experience on *X* field. They might miss something indeed. But it’s better not to presume they don’t do the research before submitting the patches. It makes no sense and it's condescending to *underestimate* someone’s skills simply because they has no prior experience on one field. Wisdom is not proportional to the experience or seniority.

Sometimes, the contemptuous attitude can hide behind some positive words trickily. For example, the comments like "your code is better than what I expect" (It’s worse when the author is female, person of color, and/or young but actually experienced) or "I empower you the freedom to structure the code in a way you like" (They are born free and have natural right to shape their code in any style) are nonsensical and arrogant. Underestimating someone is unacceptable. Talking down to others is uneducated. I try my best to avoid the comments that might be misunderstood like above and offend others. As an Asian in the U.S., I know how *microaggression* can hurt.

I think this is the most important part when doing a review. The tone and the attitude can seriously impact the interactions during the review. The reviewers should collaborate with the code author **as equals**, rather than imagining the code author knows less.

As long as the planned requirements are met and the patch makes the codebase better, I will accept the patch. Even the author’s style is different from my one. If there is something unexpected but without logical defects, I will ask why instead of asking them to change the code. The only thing reviewer needs to do is helping the code auther to achieve the planned requirements (e.g., planned features or bug fixing) and catch the logical defects in their code. Personal preferences, emotions, and feelings are not factors to block the review. Being rational is one of the key properties of a good reviewer.

Additionally, I like to do the research around the topic I have questions about with the code author together. I won’t ask the code author to collect all the information needed by themselves then sit and wait there to make a call. I believe doing the research is part of the job for doing a review. We are working together to make the code better.

### Review Focus

> “If I had an hour to solve a problem
> I’d spend 55 minutes thinking about the problem
> and 5 minutes thinking about solutions.”
>
> ― Albert Einstein

I focus more on the *problem* than the *solution*.

I believe how well we define a problem determines how well we solve it. I focus on what the problem exactly is and why and when the problem occurs. The deeper we dig into the problem, the higher chance we can find out the root cause and the appropriate requirements. There are always more than one approaches to solve the problem. But it’s impossible to find the most appropriate solution without analysing the problem elaborately. I tend to ask what the problem is, or why it occurs, before reviewing the code detailedly (one example [here](https://github.com/kinetiknz/cubeb/pull/597#issuecomment-663144179)). The syntax, style, or code-pattern is less important. After all, doing the right thing is more important than doing things right.

Challenging assumptions, considering a broader or narrower version of the problem, or rephrasing the problems are common ways to look at the problem from different perspectives.

## Code Reviews in Mozilla

To be clear, I am not expressing the following code is bad. Most of the time my teammates’ code is generally good! The following list is only used to prove I do my daily job conscientiously and I don’t live up to my teammates’ trust. They are all awesome engineers. They’ll be happier if I can catch the error before it’s shipped to the users ends.

### Components

- cubeb: The cross-platform audio library of Firefox
  - cubeb-coreaudio: The cubeb implementation on MacOS via `AudioUnit` APIs
  - cubeb-pulse: The cubeb implementation on Linux via `libpulseaudio` APIs
- audio-ipc: The IPC (inter-process communication) protocal between main (parent) process and content (child) process within Firefox
- autoplay: HTML `<video>` autoplay mechanism
- media-key-control: media-key control mechanism within Firefox that achieves [W3C media-session API](https://w3c.github.io/mediasession/)
- mp3-demuxer: The demuxer for MP3 format

### List

1. I have sharp eyes to catch algorithmic defects or logic conflicts
   - *media-key-control* - [BMO 1654657](https://phabricator.services.mozilla.com/D85514?id=319982#inline-491358): Catch an out-of-order problem when updating main-media-controller
   - *media-key-control* - [BMO 1592454](https://phabricator.services.mozilla.com/D54388?id=198735#anchor-inline-339178): Catch an out-of-order problem when updating the active media session. This is critical since it leads to re-implement all the things
   - *media-key-control* - [BMO 1633565](https://phabricator.services.mozilla.com/D73374#inline-440627): Catch an architectural defect that stops us to implement the expected behavior.
   - *media-key-control* - [BMO 1593131](https://phabricator.services.mozilla.com/D51766?vs=on&id=186467#anchor-inline-315773): Catch an error that the keypress events are monitored only for a short period of time
   - *media-key-control* - [BMO 1577367](https://phabricator.services.mozilla.com/D56514#anchor-inline-343530): Catch an logical defect that leads to re-architect the module in [BMO 1571493](https://phabricator.services.mozilla.com/D57574?id=210756#inline-353340)
   - *media-key-control* - [BMO 1584501](https://phabricator.services.mozilla.com/D47546?id=175094#inline-299094): Catch an error that initialization return success but it may fail actually
   - *media-key-control* - [BMO 1623486](https://phabricator.services.mozilla.com/D67712?id=248138#inline-405706): Catch [BMO 1626600](https://bugzilla.mozilla.org/show_bug.cgi?id=1626600) when reviewing patches
   - *cubeb-pulse* - [#63](https://github.com/djg/cubeb-pulse-rs/pull/63#pullrequestreview-510856014): Catch an out-of-sync problem between two related variables
2. I am able to reduce the code complexity by suggesting a better algorithm or data structure
   - *media-key-control* - [BMO 1633010](https://phabricator.services.mozilla.com/D73486#inline-433818): Apply logic reasoning to restrict the object lifetime and avoid abusing the shared-pointers
   - *media-key-control* - [BMO 1611332](https://phabricator.services.mozilla.com/D60935?vs=223340&id=223691#inline-371097): Propose an algorithm to centralize one specific job that done in different functions to one place, by leveraging a logical fact of the in-use hashtable
     - The comments aren’t updated since I explained how this logic works face-to-face with the author
   - *media-key-control* - [BMO 1654657](https://phabricator.services.mozilla.com/D85514?id=323396#inline-495158): Suggest an O(1) algorithm instead of the O(N) approaches used in the patch
     - But it’s ok to leave the O(N) as it is
   - *audio-ipc* - [#107](https://github.com/djg/audioipc-2/pull/107#discussion_r507953026): Reduce a redundant variable whose value can be inferred by another variable
   - *cubeb-coreaudio* - [#86](https://github.com/ChunMinChang/cubeb-coreaudio-rs/pull/86) / [BMO 1629839](https://bugzilla.mozilla.org/show_bug.cgi?id=1629839#c2): Simplify the proposed solution
3. I am able to provide a better code pattern or logic dependencies in code, to avoid the foreseeable errors
   - *autoplay* - [BMO 1578615](https://phabricator.services.mozilla.com/D44746#inline-285079): Ask the author to apply the [factory function](https://abseil.io/tips/42) style to stop calling a method by an object when the object initialization fails
   - *media-key-control* - [BMO 1633010](https://phabricator.services.mozilla.com/D73486?id=273949#inline-433803): Ask the author to make sure certain logical relationships between some variables. That catches a [bug here](https://bugzilla.mozilla.org/show_bug.cgi?id=1627999#c16). (The [fix is here](https://phabricator.services.mozilla.com/D75477))
   - *media-key-control* - [BMO 1584501](https://phabricator.services.mozilla.com/D47546?id=175094#1492850): Ask the author to apply the [factory function](https://abseil.io/tips/42) style to stop calling a method by an object when the object initialization fails
4. I am able to foresee the problem from a spec before and without implementing the code
   - W3C media session issue [#254](https://github.com/w3c/mediasession/issues/254): [BMO 1621403](https://phabricator.services.mozilla.com/D82816?id=310984#inline-476211)
5. I can review the math calculation carefully
   - *mp3-demuxer* - [BMO 1423834](https://bugzilla.mozilla.org/show_bug.cgi?id=1423834): Catch and [prove](https://chunminchang.github.io/blog/post/running-average) the average calculation is wrong when having a look at the code 
   - *cubeb-coreaudio* -  [private #1](https://github.com/padenot/cubeb-coreaudio-rs-private/pull/1#discussion_r401092469): Correct the calculation of the audio samples. Incorrect calculation can damage the audio quality.
   - *cubeb-coreaudio* -  [#99](https://github.com/ChunMinChang/cubeb-coreaudio-rs/pull/99#discussion_r459775374): Make sure all the latency is counted in a total-latency-time
   - *cubeb-coreaudio* -  [#101](https://github.com/ChunMinChang/cubeb-coreaudio-rs/pull/101#discussion_r459078356): Suggest having a default value for a variable whose value can be changed on the fly. That change would fit in more general cases
   - *cubeb-coreaudio* -  [#107](https://github.com/ChunMinChang/cubeb-coreaudio-rs/pull/107#discussion_r461091182): Make sure the bound of stream position is correct
   - *cubeb-pulse* - [#63](https://github.com/djg/cubeb-pulse-rs/pull/63#pullrequestreview-510856014): Correct the calculation of the audio samples
6. I am able to catch potential crash/UAF/data race
   - *media-key-control* - [BMO 1611332](https://phabricator.services.mozilla.com/D60935?id=222398#inline-370531) ([BMO 1621779](https://bugzilla.mozilla.org/show_bug.cgi?id=1621779)). Catch a UAF. The right fix lead us to find [BMO 1623950](https://bugzilla.mozilla.org/show_bug.cgi?id=1623950)
   - *media-key-control* - [BMO 1633010](https://phabricator.services.mozilla.com/D73486?id=271074#inline-431599) Catch a data race issue between shutdown and dispatching key-event
   - *audio-ipc* - [#62](https://reviewable.io/reviews/djg/audioipc-2/62#-LjLm3ei63coXTWilJ0C): (crash) Catch a crash that can happen when device collection changes
   - *media-key-control* - [BMO 1634494](https://phabricator.services.mozilla.com/D87143?id=330905#inline-502409): (crash) Catch a crash that can happen when media-sesson is not set but some specific media key is pressed 
7. I am able to maintain/improve the code readability
   - *audio-ipc* - [#112](https://github.com/mozilla/audioipc-2/pull/112/files#r569712579): Ask author to add a comment indicating the message-channel recevicer takes a message-channel error as the signal to execute task, which is an untraditional way of using message-channel sender and recevicer.
   - *cubeb* - [#463](https://github.com/kinetiknz/cubeb/pull/463#discussion_r223819871): Replace `if` that always work by `assert` and remove the misleading comments
   - *media-key-control* - [BMO 1627999](https://phabricator.services.mozilla.com/D75477?id=279333#inline-438887): Ask the author to add an assertion to let the reader know the current code state
  
There should be more reviews I did. But the above are what I can remember for now.

## Other Reviews and Communications

I am also willing to raise my opinions outside of Mozilla. When implementing the media-session, I filed several [spec issues](https://github.com/w3c/mediasession/issues?q=is%3Aissue+author%3AChunMinChang) and proposed [potential solutions](https://github.com/w3c/mediasession/issues/234#issuecomment-530540930).

The team-wise communication is a important part to make the whole team succeed. I am willing to help the team members to get the information they need to take further actions, or share my views to any problem I spot. For example, I reported a *WebRTC* device-switching bug and gave a [code analysis about the cause and its possible solutions](https://bugzilla.mozilla.org/show_bug.cgi?id=1572281#c2) even though *WebRTC* is not the area I was working on. I also let the *mp4parse-rust* contributors noticed their new test settings [cannot work with Rust Nightly](https://github.com/mozilla/mp4parse-rust/pull/205#issuecomment-597326901) so they can fix it.

## Closing Words

I hope the review-records here can reveal a part of my daily job. It’s a reference to myself and to anyone who is interested in my review skills. Code-review is an important part to help the whole team succeed. Besides my daily programming work, I try my best to help others.

During the pademic, I come to realize what mindset I need to have when reviewing the patches from my peers. I come to realize that everyone needs companionship and constructive suggestions -- Cut right to the point, but “you don’t have to face the difficulty alone”.

[overengineering]: why-is-overengineering-bad
[specific-code-is-bad]: avoid-adding-code-for-a-specific-case-only