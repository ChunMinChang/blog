---
layout: post
title: 'Firefox WebRTC: Multi microphones'
category:
- Work
tags:
- Firefox
- Media
comments: true
type: photo
imagefeature: "../images/posts/multi-mics/multi-mics-icon.png"
image: "../images/posts/multi-mics/multi-mics-icon.png"
date: 2022-04-19 07:44 +0800
---

Firefox Nightly now allows users to use as many microphones as they want at the same time during video conferencing!

The most important benefit is that you can easily switch your microphones at any time, if your conferencing service provider enables this flexibility!

<!--read more-->

## Multi-microphones in Firefox

Now you can open more than one microphone at a time via `getUserMedia` in Firefox Nightly! The [`getUserMedia`][gUM] is an interface that prompts the user for permission to open a media input, which will produce media data for the web service.

In the past, Firefox could use only one microphone per [window][window] most of the time. Now the restriction is lifted! The example code below can demonstrate the difference between the *before* and *after*.

```js
function runTest() {
  let devices = await navigator.mediaDevices.enumerateDevices();
  // Create constraints
  let constraints = [];
  devices.forEach((device) => {
    if (device.kind === "audioinput") {
      constraints.push({
        audio: { deviceId: { exact: device.deviceId } },
      });
    }
  });

  if (constraints.length < 2) {
    console.log("Need at least 2 audio devices!");
    return;
  }

  try {
    // Open microphones by the constraints
    let mediaStreams = [];
    for (let c of constraints) {
      let stream = await navigator.mediaDevices.getUserMedia(c);
      console.log(
        "MediaStream: " +
          stream.id +
          " for device: " +
          c.audio.deviceId.exact +
          " is created"
      );
      mediaStreams.push(stream);
    }
    // Close microphones
    for (let stream of mediaStreams) {
      for (let track of stream.getTracks()) {
        console.log("Stopping " + track.label);
        track.stop();
      }
      console.log("Stop all tracks in MediaStream: " + stream.id);
    }
    mediaStreams = [];
  } catch (e) {
    console.log(e.name + ": " + e.message);
  }
}
```

Before the constraint has been removed, the `runTest` above will get a `NotReadableError`:
![before][before]

After the limitation has been freed, the `runTest` above will open and then close all the microphones once the user permits Firefox to open them:
![after][after]

## Demos

You can see the online demo of the above code [here][example]. Aside from it, [here][chunmin-demo] is a test page that routes the captured audio from the chosen microphones to an HTML `<audio>` element. [Here][padenot-demo] is another test page powered by `WebAudio` APIs that shows the volumes of all the selected microphones.

## Benefits

The most beneficial change is that this makes microphone switching easier!

When the user asks to switch a microphone during video conferencing, the service websites usually open a second/new microphone to check the volume and other audio settings before closing the first/original one. To work around the constraint Firefox used to have, the service websites may need extra work, e.g., opening the second/new microphone in an iframe, which usually wastes more resources and power.

After the restriction is lifted, there is no reason to do those workarounds anymore. The service website can easily enable the second or third microphone via the `getUserMedia` as they do for the first one. In addition, users can use multiple microphones simultaneously during the video conferencing or in some WebAudio apps if the video conferencing website adopts this feature in their services.

## Lessons learned

Below is my development story and what I've learned from this project.

### Review the plan carefully

I should do more research and ask more detailed review of my design before implementing the code.

Enabling multi-microphone in Firefox is my first WebRTC project. WebRTC is a whole new world to me. Since I had never worked in WebRTC code before, I took one week or two to do the research and wrote down a design document for this project, with the block diagram of the code modules. However, the lack of domain knowledge leads me to underestimate the project's scope and have a less-optimal design. The worst of it is that I realized this after implementing the whole code. In the end, I revised my design and re-do most of all my implementations.

### Comments are indispensable

The comments for the design concept and goal are necessary, especially the calculations for a particular purpose.

I am not the type of person who adds comments everywhere. I believe the code can explain itself well by using proper names for variables and functions, appropriate function size..., etc (see more in [*Cleaning Code*](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)). But it's vital to add comments if the purpose is not clear. The code may be able to explain what they are doing itself. But sometimes, it's hard to see why they do things in a certain way.

When I researched how to split the code into different modules, I found some mysterious calculations that append and remove audio data in certain conditions (and sometimes it could cause crashes). It seems to me it's a workaround of something. But without understandable comments and fair enough domain knowledge, I don't know why and when it needs to do that calculation. Except for the original code author, no one in our team knows what it is, or they simply forgot it. To make matters worse, the author was taking a long leave. So I decided to move all those codes into the same module and leave them as a block box.

Unfortunately, that module is the essential building block of the architecture. I realized that I needed to have a new design for the whole project after I had a chance to discuss it with the original author. I needed to replace that workaround with a better fix.

In the end, I took a different approach with the same concept to solve the problem that the code intended to solve. The most important difference I made is that I added a [short paragraph][bmo1741959-comment] explaining the issue we faced and the design concept with a mathematical proof. I hope those comments can help others understand the intention of that code better, so they can modify the code without asking for my help.

### Testability without dependencies

Make sure you can test every single module without other dependencies.

I consistently applied *TDD* (Test Driven Development) when possible. I like to write high-level integration tests that illustrate the project's main goal before working on the production code and gradually adding unit tests while developing.

To enhance my test approach in this project, I used [dependency injection][DI] techniques to isolate code modules in my unit tests, mainly *constructor injection*. [*gMock*][gmock] is a practical tool/framework  that makes this easy for me. Every C++ programmer should know how to use [*gTest*][gtest], and every *gTest* user should use *gMock* for unit tests in isolation.

### Breaks down a problem into several sub-problems

Break down the task into smaller sub-tasks by the dependencies.

It's always good to split the task into several smaller ones and finish them individually. I didn't always do this since it's easier for me to manage the related code patches in one git branch. More precisely, I did break down the task into smaller chunks and finished them separately. However, I tended to fix several problems in one individual task.

Now I prefer to fix the problems one by one: one problem per task. The benefit is that it forces me to outline the problem dependencies. It makes sketching the project's scope easier and calculating how many milestones the project might have possible. By having a smaller task size, I can land the patches reaching the reviewers' agreements sooner. The sooner I can test them in the wild, the sooner I can find out the problems. In addition, it gives me a positive feeling since I can see I am making progress, step by step.

I changed my task-dividing approach when I realized that I underestimated the project's scope. It probably doesn't matter if I fix all the problems or implement all the code in one bug ticket for a small project. But for a large project like this, I chose to break down the goal into smaller ones than I usually did. Besides the benefits mentioned above, the smaller task also lowers the brain burden for my code reviewers since they will only see the code they need to know. It's easier for me to multitask by communicating with a different person individually.

## Other engineering details

The following are some notes for myself (in case I need to revisit this project) or others who have an interest in contributing to our codebase.

### Architecture

The main goal of this project is to enable [`MediaTrackGraph`][MTG] (*MTG*) to have multiple audio streams and the ability to process them. `MediaTrackGraph` (*MTG*) is Firefox/gecko's internal WebRTC data processing framework.

#### Original design

![original][basic]

Each *MTG* has only one audio stream in the original design, whose periodical callback gives the [`AudioCallbackDriver`][ACD] (`GraphDriver`) signals to drive the *MTG* to process the audio data periodically. In other words, the underlying audio callback propels the *MTG* to do the audio processing in a constant cycle. The primary purpose of this design is to make sure *MTG* can process the audio data in real-time.

When the user asks to use a particular audio device, *MTG* will take two actions:
1. Create an `AudioCallbackDriver` (`GraphDriver`), and then `AudioCallbackDriver` will start an audio stream capturing the user-selected device's sound.
2. Create [`AudioInputTrack`][AIT] processing the data from the underlying audio stream through `AudioCallbackDriver` and *MTG*. Each `AudioInputTrack` pairs a [`MediaStreamTrack`][MST], the javascript interface to operate the audio.

Once the underlying audio stream kicks off, it periodically pushes the captured voice to `AudioInputTrack` through `AudioCallbackDriver` and *MTG*. Next, *MTG* will start an audio processing iteration to process the data stored in the `AudioInputTrack`.

#### Problems

The limitation of this design is it only allows one audio stream per *MTG*. To make *MTG* have multiple audio streams, it needs to:
1. have a way to manage the audio stream in other places than `AudioCallbackDriver`
2. find a way to route the data of a non-driver-owned audio stream to the *MTG*.

How hard would it be? Sounds simple, right?

There are several problems with achieving this:

1. `AudioInputTrack` is tied to `AudioCallbackDriver`. `AudioInputTrack` stores the audio data from the device, but the data is from `AudioCallbackDriver` instead of the audio stream directly. It needs a new way to get the audio data from a non-driver-owned audio stream.
2. Each audio stream will produce its data on its own threads. To make sure *MTG* process the data in real-time, it needs a lock-free/mutex-free technique to read and write the audio data.
3. Different audio devices run on different clocks with varying rates of sampling. It needs some calculations to align the audio data in different sampling rates.
4. Users can stop the audio device at any time. Suppose the driver-owned audio stream stops before the non-driver-owned audio stream. Then there're two problems:
   1. `AudioCallbackDriver` loses its power source to drive the *MTG* (we could use an output-only audio stream, but there is no benefit. See below).
   2. There is no central clock for the non-driver-owned audio streams to align with anymore (we could use a default rate, but we will sacrifice the audio quality caused by the clock-alignment calculation for nothing). In that case, one of the non-driver-owned audio streams should become a new main audio stream whose clock is the new standard.

#### New design

![new][design]

We've developed some essential modules to address part of the problems above over time in different projects. It comes to the point of reworking Firefox/gecko's WebRTC infrastructure. Hence, I redesigned the architecture illustrated in the above figure.

The data processing procedure is the same as the original design. The key difference is that the `AudioInputTrack` is replaced by `AudioProcessingTrack` and [`NativeInputTrack`][NIT] or [`NonNativeInputTrack`][NNIT].

The `NativeInputTrack` is where to read the driver-owned audio's data, while the `NonNativeInputTrack` is where to read the non-driver-owned audio's one. They inherit the same class named [`DeviceInputTrack`][DIT], so the *MTG* can start, stop, and read data from the audio stream via the same interface, whether the track is driver-owned. The *native* refers to the driver-owned audio, and the *non-native* refers to the non-driver-owned one. *MTG* manages all the `DeviceInputTrack`s. Every audio device has a corresponding `DeviceInputTrack`, which stores the data captured from the device, shared with several `AudioProcessingTrack`s.

The [`SPSCQueue`][spscq] is a single-producer-single-consumer lock-free data structure that writes data on a particular thread and reads it on another. The non-native (non-driver-owned) audio callback thread will periodically write audio data into the `SPSCQueue`; *MTG*'s graph thread will read the `SPSCQueue` in each audio processing iteration. The data read from the `NonNativeInputTrack` will be aligned with the one read from `NativeInputTrack` in the [*drift correction*][ADC] module before those data flow to the `AudioProcessingTrack`, where to do the audio processing.

When the user stops/closes the native audio device, which is the device for the native (driver-owned) audio stream, *MTG* will promote one of the non-native (non-driver-owned) audio to a native (driver-owned) one. In other words, *MTG* will create a new `NativeInputTrack` for the selected device and destroy the `NonNativeInputTrack` the selected device used to pair. If there are several non-native audio devices, the earliest started/opened non-native audio device will be [chosen][fofp] (first-open-first-promote).

Note that the non-native audio stream is created within a *task thread* rather than the *graph thread*. The reason is that creating an audio stream within another audio stream's callback thread in some of our audio backends (e.g., [PulseAudio][PA]) leads to a deadlock. In some cases, the graph thread is the same as `AudioCallbackDriver`'s thread, the native audio callback thread. Therefore, creating an audio stream (for non-native audio) on the *graph thread* can lead to a deadlock. We use the same task thread for all the audio stream operations for now to avoid the unnecessary or potential racing issues that might happen on different audio backends.

The actual code module design is more complicated than the figure above. But a simple-enough figure can explain the idea better.

### Ownerships

![ownership][ownership]

There are several classes between `NonNativeInputTrack` and its underlying audio stream in the current implementation. [`AudioInputSource`][ais] is a class operating the audio stream within a task thread and ref-countable since both the graph thread and the task thread will use it. [`CubebInputStream`][cis] is a C++ friendly wrapper for cubeb C APIs and is owned by the `AudioInputSource` alone.

`NonNativeInputTrack` owns an `AudioInputSource`, created by `MediaTrackGraph`, once the user asks to start non-native audio on the graph thread. The stream creation, start and stop are executed on the task thread within a [`Runnable` task][ais-start] that keeps a reference to `AudioInputSource` during its task execution. `AudioInputSource` holds a reference to its owner `NonNativeInputTrack`, so it knows where to forward the events from the underlying audio stream. When the user no longer needs the audio, `NonNativeInputTrack` drops its `AudioInputSource`, on the graph thread, after asking the task thread to execute the stream stop operation. Once finishing the task, no one keeps the reference to `AudioInputSource`, so it will destroy itself and remove its reference to its owner `NonNativeInputTrack`.

I hope this can give you, probably future me, a rough idea about the ownerships between the classes. However, the above description might not hold for too long. The code is likely to be updated from time to time. It's better to read our current code to know the latest development.

## Closing Words

It is an unexpectedly arduous journey. I enjoyed the thought process of shaping a raw idea into a proper design and then implementing it. It brings back my memory when working on [cubeb oxidation][cubeb-oxidation].

It took me almost a year from starting to finishing it, from learning WebRTC code to implementing all the code alone. Fortunately, I have enough support when needed, and my reviewers are always willing to take their time to discuss with me.

I have mixed feelings about this project. I didn't expect it would take me so long to accomplish the mission. I felt terrible when things weren't going well. I felt annoyed when digging into problems in unfamiliar code. On the other hand, I appreciate this project gives me a chance to learn how to calm myself down and enrich my engineering experience in a new domain, which will advance my skills in the long run.

Life is full of ups and downs, so make the project progress. I've learned how to keep moving forward between the rises and falls. The experience I gained from this project will help me conquer my next project and all the followings. I have faith in myself. I believe I can overcome my upcoming challenges, and it will become another unforgettable memory.

[gUM]: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia
[window]: https://developer.mozilla.org/en-US/docs/Web/API/Window

[before]: ../images/posts/multi-mics/error.png "Get a NotReadableError"
[after]: ../images/posts/multi-mics/success.png "Successfully open multi microphones"

[example]: https://chunminchang.github.io/playground/webrtc/all_microphones.html
[chunmin-demo]: https://chunminchang.github.io/playground/webrtc/multi_microphones.html
[padenot-demo]: https://paul.cx/public/testing-multi-gum.html

[bmo1741959-comment]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/webrtc/MediaEngineWebRTCAudio.cpp#L558-L666
[bmo1238038]: https://bugzilla.mozilla.org/show_bug.cgi?id=1238038

[cubeb-oxidation]: summary-of-cubeb-oxidation-on-mac-os

[DI]: https://en.wikipedia.org/wiki/Dependency_injection
[gmock]: http://google.github.io/googletest/gmock_for_dummies.html
[gtest]: https://google.github.io/googletest/

[basic]: ../images/posts/multi-mics/basic.png "Original architecture"
[design]: ../images/posts/multi-mics/design.png "New architecture"
[ownership]: ../images/posts/multi-mics/ownership.png "Ownerships"

[MTG]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/MediaTrackGraph.h#L55-L87
[ACD]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/GraphDriver.h#L544-L563
[AIT]: https://github.com/mozilla/gecko-dev/blob/81d623f7fc5f4e9acbe31203bc9a0868078dbe09/dom/media/webrtc/MediaEngineWebRTCAudio.h#L253
[MST]: https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack
[DIT]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/DeviceInputTrack.h#L102
[NIT]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/DeviceInputTrack.h#L210
[NNIT]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/DeviceInputTrack.h#L245
[ADC]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/AudioDriftCorrection.h
[spscq]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/mfbt/SPSCQueue.h#L65-L92
[fofp]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/MediaTrackGraph.cpp#L4054
[ais]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/AudioInputSource.h#L21-L27
[ais-start]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/AudioInputSource.cpp#L78
[cis]: https://github.com/mozilla/gecko-dev/blob/6da1ebe13b260efabd88eb98dec5fa8ee65987b2/dom/media/CubebInputStream.h#L18-L21

[PA]: https://www.freedesktop.org/wiki/Software/PulseAudio/
