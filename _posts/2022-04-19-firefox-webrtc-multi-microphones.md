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

The most important benefit is that you can switch your microphones more easily during a call, if your conferencing service provider enables this flexibility! (e.g, [jitsi PR 1988](https://github.com/jitsi/lib-jitsi-meet/pull/1988))

<!--read more-->

## Multi-microphones in Firefox

Now you can open more than one microphone at a time via `getUserMedia` in Firefox Nightly! The [`getUserMedia`][gUM] is an interface that prompts the user for permission to open a media input, which will produce media data for the web service.

In the past, Firefox could use only one microphone per [window][window]. Now the restriction is lifted! The example code below can demonstrate the difference between the *before* and *after*.

```js
async function runTest() {
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

When the user asks to switch a microphone during video conferencing, the service websites usually open a second/new microphone to check the volume and other audio settings before closing the first/original one. To work around the constraint Firefox used to have, the service websites may need extra works, e.g., opening the second/new microphone in an iframe, which usually wastes more resources and power.

```js
async function runTest() {
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
    // Open the audio input device
    let c1 = constraints.shift();
    console.log("Open audio input: " + c1.audio.deviceId.exact);
    let stream1 = await navigator.mediaDevices.getUserMedia(c1);
    console.log("MediaStream: " + stream1.id + " is created");

    // Switch the audio input device
    let c2 = constraints.shift();
    console.log(
      "Switch audio input from: " +
        c1.audio.deviceId.exact +
        " to: " +
        c2.audio.deviceId.exact
    );
    let stream2 = await navigator.mediaDevices.getUserMedia(c2);
    console.log("MediaStream: " + stream2.id + " is created");
    for (let track of stream1.getTracks()) {
      console.log("Stopping " + track.label);
      track.stop();
    }
    console.log("Stop all tracks in MediaStream: " + stream1.id);

    // Close the audio input device
    for (let track of stream2.getTracks()) {
      console.log("Stopping " + track.label);
      track.stop();
    }
    console.log("Stop all tracks in MediaStream: " + stream2.id);
  } catch (e) {
    console.log(e.name + ": " + e.message);
  }
}
```

After the restriction is lifted, there is no reason to do those workarounds anymore. The service website can easily enable the second or third microphone via the `getUserMedia` as they do for the first one. In addition, users can use multiple microphones simultaneously during the video conferencing or in some WebAudio apps if the video conferencing website adopts this feature in their services.

## Lessons learned

Althoug I am quite content with my outcomes (Aug, 08, 2023 update: no regression found yet), I don't have the same feeling about my journey.

The developement journey was unexpectedly longer than what I expected. Battling in an unfamiliar field with a mission is quite struggling. I learned so much things in a hard way:

1. First of all, I should do more research and ask more detailed review of my design before implementing the code.
2. Second, I found that the comments for the design concept and goal are indispensable, especially for the calculations in some particular cases.
3. Third, testing is key to make sure every module works and guarantees those modules can work together.
4. Finally, breaking down the problems, and then fixing them and shiping them one by one, is better than shipping them all at once.

I am going to detail each of the points below.

### Review the plan carefully

First of all, I should do more research and ask more detailed review of my design before implementing the code.

Enabling multi-microphone in Firefox was my first WebRTC project. WebRTC is a whole new world to me. Since I had never worked in WebRTC code before, I took one week or two to do the research and wrote down a design document for this project, with the block diagram of the code modules. However, the lack of domain knowledge led me to underestimate the project's scope and had a less-optimal design. The worst of it is that I realized this after implementing the whole code. In the end, I revised my design and re-do most of all my implementations.

I might be able to find a better solution earlier if I can study more, to have more engineering details to discuss with the right person.

### Comments are indispensable

Second, the comments for the design concept and goal are indispensable, especially for the calculations in some particular cases.

I am not the type of person who adds comments everywhere. I believe the code can explain itself well by using proper names for variables and functions, appropriate function size..., etc (see more in [*Cleaning Code*](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)), but it's vital to add comments if the purpose is not clear. The code may be able to explain what they are doing itself. However, it's hard to see why they do things in a certain way sometimes.

When I studied how to split the code into different modules, I found some mysterious calculations that append and remove audio data in certain conditions (and sometimes it could cause crashes). It seems to me it's a workaround of something. Yet, without understandable comments and fair enough domain knowledge, I didn't know why and when it needs to do that calculation. Except for the original code author, no one in our team knew what it is, or they simply forgot it. To make matters worse, the original author was taking a long leave. Thus, I decided to move all those codes into the same module and leave them as a black box.

Unfortunately, that black box is the essential building block of the architecture. I realized that I needed to replace that black box with a proper fix after I had a chance to discuss it with the original author. This requirement made me have a new design for the whole project. Due to the changes, this project took longer time than I expected.

In the end, the problem was solved by a new method with the same concept, and I also **mathematically** prove it works. I added a [short paragraph][bmo1741959-comment] explaining the issue we faced and the design concept with the proof. The explanation and the proof should help the readers understand how the code works. With a clear comments, the calculation is no longer a puzzle that no one can understand it.

### Testability without dependencies

Third, testing is key to make sure every module works and guarantees those modules can work together.

I consistently applied *TDD* (Test Driven Development) when possible. I like to write high-level integration tests that illustrate the project's main goal before working on the production code and gradually adding unit tests while developing.

To enhance my test approach in this project, I used [dependency injection][DI] techniques to isolate code modules in my unit tests, mainly *constructor injection*. This way, the module can be tested **without other dependencies**. [*gMock*][gmock] is a practical tool/framework that makes this easy for me. Every C++ programmer should know how to use [*gTest*][gtest], and every *gTest* user should use *gMock* for unit tests in isolation.

With the tests, I was able to catch the problem in an early stage.

### Breaks a problem down into several sub-problems

Finally, breaking down the problems, and then fixing them and shiping them one by one, is better than shipping them all at once.

Breaking down the problem really depends on the experience. The same problem can be divided into different sets of sub-problems in different views. However, once the task is divided into several smaller sub-tasks, the most important thing to do is to sort out their dependencies. Those dependencies are the keys to decide what problems need to be solved first. With the order of the divided sub tasks, we are able to fix and ship each of them individually, and finish the whole project gradually over the time.

While spending time to divide patches sounds like a waste of energy, breaking the problem down into several sub tasks and address them one by one can provide several benefits.

1. Having sub tasks makes sketching the project's scope easier and help calculating how many milestones the project have.
2. It's easier to work on a smaller problem, for both code authoer and reviewers. It's easier to discuss one issue at a time.
3. It's easier to multitask on different sub-problems. We can work on several problems that don't have dependencies on each other at a time.
communicating with a different person individually.
4. The step-by-step progress can be visualized, and it can give a positive feeling!

To be honest, this is the hardest habit for me to establish.

## Other engineering details

Now, let's talk about some engineering details. This is pretty Firefox/Gecko-specific, so if you're not interested, feel free to skip this part.

The following are some notes for myself (in case I need to revisit this project) or others who have an interest in contributing to our codebase.

### Architecture

The main goal of this project is to enable [`MediaTrackGraph`][MTG] (*MTG*) to have multiple audio streams and the ability to process them. `MediaTrackGraph` (*MTG*) is Firefox/Gecko's internal WebRTC data processing framework.

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

We've developed some essential modules to address part of the problems above over time in different projects. It comes to the point of reworking Firefox/Gecko's WebRTC infrastructure. Hence, I redesigned the architecture illustrated in the above figure.

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

### Audio Processing

In order to do audio processing like noise suppression or echo cancellation for the voice data from microphone, the [`AudioProcessing` module in `libwebrtc`][libwebrtc-audioprocessing] is integrated into our audio processing pipeline to do the job.

This audio processing module needs a *specific amount* of data to do the processing. It requires its input to be a **[10-ms][libwebrtc-audioprocessing-10ms]** chunk of voice data. However, the input of our audio pipeline is not always guaranteed to have data with the times of 10ms length. For example, we may have 15ms of data, which means we can only feed the first 10ms of data and leave the 5ms of data unprocessed. If we only have 8 ms of data, then no data will be processed.

To force processing the input data, a naive way is to append silent data when there is no enough data to process. That is, *x* ms of silent data will be appended when data is *(10-x)*-ms length. Nevertheless, this could result in **glitch** sounds in the audio.

Instead of inserting the silent data between the voice data, we can append the silence in advance, before all the data, so there won't be any glitch. The only drawback is the real voice data won't be sent out immediately. The pre-buffered silent data will be sent as the output first. But how many silent data we need to feed in advance? The answer is **at least 10ms** of data!

More specifically, the design is to have an *internal buffer* that pre-loads at least 10ms of silent data in advance. The *internal buffer* has three components, *pending*, *audio-processing* and *ready*. *Pending* holds the *unprocessed* data that will be sent to *audio-processing*, *ready* stores the *processed* data out of *audio-processing*, while *audio-processing* is the place that takes raw voice input data, do audio processing, and then return the processed data out.

```
        +------------------------------------------------+
        |   +---------+                       +-------+  |
input --+-->| pending |--> AudioProcessing -->| ready |--+--> output
        |   +---------+                       +-------+  |
        +------------------------------------------------+

        < --------------- internal buffer --------------->
```

In the design, the *internal buffer* will output the same amout of data as input, and the silent data are pre-buffered in *ready*.

When the input data is less than *10ms*, we stock the input data in *pending* and return the buffered data in *ready* as output. That's said, suppose we pre-buffer 10ms of silent data in *ready*. If the input has only 3ms of data, those data will be stocked in *pending*. Then, *ready* will move 3ms of the buffered data as the output, and leave 7ms of data inside.

Next, if the new input makes *pending* have enough data to process, we forward as much as *10ms* chunks as we can to *audio-processing*, and move the processed results to *ready*. If the input at this time has 35ms of data, the first 7ms of data of the input with the 3ms data left in *pending* will result in a 10ms chunk that can be passed to *audio-processing*. Then, the *audio-processing* will return 10ms of processed data to *ready*. Next, following the same process, we can take 20ms of data from input and produce 20ms of processed data to *ready*. At the end, the left 8ms of input data will be stocked in *pending* since they are not long enough to be a valid input of *audio-processing*.

Now, we've produced 30ms of processed data to *ready*, and make *ready* have 37ms of data. After moving 35ms of data from as output, *ready* will leave 2ms of data inside. As a result, we will still leave 10ms data in the *internal buffer*: 8ms of data in *pending*; 2ms of data in *ready*. Repeating this process guarantees that *internal buffer* always have *10ms* of data, and has no need to insert the silent data in between.

The below is the formal proof, copied from the [code comments][bmo1741959-comment] I wrote. Although the description is slighty different, the idea is same. In the description, *audio-processing* is omitted, because we only need to focus on the amount of data in *pending* and *ready*. In the comments, the *pending* referes to the *packetizer*, and the *ready* referes to *mSegment*.

```
// We will use webrtc::AudioProcessing to process the input audio data in this
// mode. The data input in webrtc::AudioProcessing needs to be a 10ms chunk,
// while the input data passed to Process() is not necessary to have times of
// 10ms-chunk length. To divide the input data into 10ms chunks,
// mPacketizerInput is introduced.
//
// We will add one 10ms-chunk silence into the internal buffer before Process()
// works. Those extra frames is called pre-buffering. It aims to avoid glitches
// we may have when producing data in mPacketizerInput. Without pre-buffering,
// when the input data length is not 10ms-times, we could end up having no
// enough output needs since mPacketizerInput would keep some input data, which
// is the remainder of the 10ms-chunk length. To force processing those data
// left in mPacketizerInput, we would need to add some extra frames to make
// mPacketizerInput produce a 10ms-chunk. For example, if the sample rate is
// 44100 Hz, then the packet-size is 441 frames. When we only have 384 input
// frames, we would need to put additional 57 frames to mPacketizerInput to
// produce a packet. However, those extra 57 frames result in a glitch sound.
//
// By adding one 10ms-chunk silence in advance to the internal buffer, we won't
// need to add extra frames between the input data no matter what data length it
// is. The only drawback is the input data won't be processed and send to output
// immediately. Process() will consume pre-buffering data for its output first.
// The below describes how it works:
//
//
//                          Process()
//               +-----------------------------+
//   input D(N)  |   +--------+   +--------+   |  output D(N)
// --------------|-->|  P(N)  |-->|  S(N)  |---|-------------->
//               |   +--------+   +--------+   |
//               |   packetizer    mSegment    |
//               +-----------------------------+
//               <------ internal buffer ------>
//
//
//   D(N): number of frames from the input and the output needs in the N round
//      Z: number of frames of a 10ms chunk(packet) in mPacketizerInput, Z >= 1
//         (if Z = 1, packetizer has no effect)
//   P(N): number of frames left in mPacketizerInput after the N round. Once the
//         frames in packetizer >= Z, packetizer will produce a packet to
//         mSegment, so P(N) = (P(N-1) + D(N)) % Z, 0 <= P(N) <= Z-1
//   S(N): number of frames left in mSegment after the N round. The input D(N)
//         frames will be passed to mPacketizerInput first, and then
//         mPacketizerInput may append some packets to mSegment, so
//         S(N) = S(N-1) + Z * floor((P(N-1) + D(N)) / Z) - D(N)
//
// At the first, we set P(0) = 0, S(0) = X, where X >= Z-1. X is the
// pre-buffering put in the internal buffer. With this settings, P(K) + S(K) = X
// always holds.
//
// Intuitively, this seems true: We put X frames in the internal buffer at
// first. If the data won't be blocked in packetizer, after the Process(), the
// internal buffer should still hold X frames since the number of frames coming
// from input is the same as the output needs. The key of having enough data for
// output needs, while the input data is piled up in packetizer, is by putting
// at least Z-1 frames as pre-buffering, since the maximum number of frames
// stuck in the packetizer before it can emit a packet is packet-size - 1.
// Otherwise, we don't have enough data for output if the new input data plus
// the data left in packetizer produces a smaller-than-10ms chunk, which will be
// left in packetizer. Thus we must have some pre-buffering frames in the
// mSegment to make up the length of the left chunk we need for output. This can
// also be told by by induction:
//   (1) This holds when K = 0
//   (2) Assume this holds when K = N: so P(N) + S(N) = X
//       => P(N) + S(N) = X >= Z-1 => S(N) >= Z-1-P(N)
//   (3) When K = N+1, we have D(N+1) input frames comes
//     a. if P(N) + D(N+1) < Z, then packetizer has no enough data for one
//        packet. No data produced by packertizer, so the mSegment now has
//        S(N) >= Z-1-P(N) frames. Output needs D(N+1) < Z-P(N) frames. So it
//        needs at most Z-P(N)-1 frames, and mSegment has enough frames for
//        output, Then, P(N+1) = P(N) + D(N+1) and S(N+1) = S(N) - D(N+1)
//        => P(N+1) + S(N+1) = P(N) + S(N) = X
//     b. if P(N) + D(N+1) = Z, then packetizer will produce one packet for
//        mSegment, so mSegment now has S(N) + Z frames. Output needs D(N+1)
//        = Z-P(N) frames. S(N) has at least Z-1-P(N)+Z >= Z-P(N) frames, since
//        Z >= 1. So mSegment has enough frames for output. Then, P(N+1) = 0 and
//        S(N+1) = S(N) + Z - D(N+1) = S(N) + P(N)
//        => P(N+1) + S(N+1) = P(N) + S(N) = X
//     c. if P(N) + D(N+1) > Z, and let P(N) + D(N+1) = q * Z + r, where q >= 1
//        and 0 <= r <= Z-1, then packetizer will produce can produce q packets
//        for mSegment. Output needs D(N+1) = q * Z - P(N) + r frames and
//        mSegment has S(N) + q * z >= q * z - P(N) + Z-1 >= q*z -P(N) + r,
//        since r <= Z-1. So mSegment has enough frames for output. Then,
//        P(N+1) = r and S(N+1) = S(N) + q * Z - D(N+1)
//         => P(N+1) + S(N+1) = S(N) + (q * Z + r - D(N+1)) =  S(N) + P(N) = X
//   => P(K) + S(K) = X always holds
//
// Since P(K) + S(K) = X and P(K) is in [0, Z-1], the S(K) is in [X-Z+1, X]
// range. In our implementation, X is set to Z so S(K) is in [1, Z].
// By the above workflow, we always have enough data for output and no extra
// frames put into packetizer. It means we don't have any glitch!
```

This was the mysterious calculations I mentioned in [*Comments are indispensable*](#comments-are-indispensable). With the clear comments, the code is more understandable and easier to maintain.

## Closing Words

Despite it is an unexpectedly arduous journey, I enjoyed the thought process of shaping a raw idea into a proper design and then implementing it. It brings back my memory when working on [cubeb oxidation][cubeb-oxidation].

It took me almost a year to finish it, from learning WebRTC code to implementing all the code alone. Fortunately, I have enough supports when needed, and my reviewers are always willing to take their time to discuss with me.

I have mixed feelings about this project. I didn't expect it would take me so long to accomplish the mission. I felt terrible when things weren't going well. I felt annoyed when digging into problems in unfamiliar code. On the other hand, I appreciate this project gave me a chance to learn how to calm myself down and enrich my engineering experience in a new domain, which will advance my skills in the long run.

Life is full of ups and downs, so making the project progress. I've learned how to keep moving forward between the rises and falls. The experience I gained from this project is going to help me conquer my next project and all the followings. I have faith in myself. I believe I can overcome my upcoming challenges, and it will become another unforgettable memory.

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

[libwebrtc-audioprocessing]: https://github.com/mozilla/gecko-dev/blob/ea11c5c0b49f9bd0fff6fa926ee95d0680af1e83/third_party/libwebrtc/modules/audio_processing/include/audio_processing.h#L54-L130

[libwebrtc-audioprocessing-10ms]: https://github.com/mozilla/gecko-dev/blob/ea11c5c0b49f9bd0fff6fa926ee95d0680af1e83/third_party/libwebrtc/modules/audio_processing/include/audio_processing.h#L83-L86
[libwebrtc-audioprocessing-10ms]: https://github.com/mozilla/gecko-dev/blob/ea11c5c0b49f9bd0fff6fa926ee95d0680af1e83/dom/media/webrtc/MediaEngineWebRTCAudio.h#L173-L178