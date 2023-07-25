---
layout: post
title: 'WebCodecs and the future of media processing on the web'
category:
- Work
tags:
- Firefox
- Media
comments: true
date: 2023-07-24 15:56 +0800
---

Today I am going to talk about three things: What's WebCodecs, its showcase, and what the future may look like with it.

<!--read more-->

## What is WebCodces

[*WebCodecs*][webcodecs-api] is a set of low-level APIs that can access and process the media data **flexibly** and **efficiently**. The flexibility means that these APIs can easily work with other APIs, such as [*WebRTC*][webrtc-api], [*WebGL*][webgl-api], and [*WebGPU*][webgpu-api]. The efficiency implies that the media data can be transferred and processed in the best performance we can offer.

### WebCodecs Interfaces

![interfaces][interfaces]

*WebCodecs* has nine interfaces: four for audio, four for video, and one for image.

On video side, `VideoFrame` represents the raw video data, and the `EncodedVideoChunk` is its encoded format. `VideoEncoder`, just like its name, is used to encoded a `VideoFrame` into a `EncodedVideoChunk`, while `VideoDecoder` is used to decode a `EncodedVideoChunk` into a `VideoFrame`.

Similarly, *WebCodecs* has symmetric interfaces on the audio side. `AudioData` represents the raw audio data, and the rest interfaces work in the same way as their counterparts.

One special thing is that *WebCodecs* has a `ImageDecoder` to decode an image into `VideoFrame`s, so the image processing can follow the pattern of video processing we are going talk later.

### Why We Need WebCodecs

But why we need *WebCodecs*? Because, in short, there is no way to access and process the media data **efficently**. Some video conferencing sites, like [*Zoom*][zoom-hack], create their own media encoders and decoders in *C* and then compile them into [*WebAssembly*][wasm] to achieve the low-latency media processing. However, this makes users waste their bandwidth to download the media encoders and decoders that are actually duplicate in the browser, just because those websites has no access to them. That's why *WebCodecs* is a better solution.

## Demo

The video here shows how `VideoDecoder` decode the `EncodedVideoChunk`s into `VideoFrame`s, and how we can process the `VideoFrame`s before painting them to the screen. In the video, we will change their colors. Let's see the demo.

[![webcodecs-demo](https://img.youtube.com/vi/sfJ1qc2d_ec/0.jpg)](https://www.youtube.com/watch?v=sfJ1qc2d_ec)

The video processing in the demo is very basic, which you can find in the textbook. But you can do more complicated processing like cartoon filter. Next, I am going to talk more about the video processing.

## Future of Media Processing

### Future of Video Processing

![video-processing][video-processing]

In the future, *WebCodecs* will be the central part linking lots of interfaces, for media processing.

The video processing in the demo can be more efficient and more magical once we build the bridges from *WebCodecs* to *WebGL* and *WebGPU*, then the processing can be accelerated by the graphics card, and processed all inside GPUs. That can boost the performance since there is no need to copy the data from GPU to CPU to process the images.

On the other hand, *WebCodecs* has a sibling spec in *WebRTC* that can stream `VideoFrame`s from your camera. So Firefox will be able to hide your video background before sending the video over the internet for better privacy.

#### Video Processing Pipeline with Streams API

![video-streams][video-streams]

We can build a pipeline to process `VideoFrame` stream by [*Streams* API][streams-api].

The `VideoFrame` stream comming from *WebRTC*, or other kind of source, can easily work with *Streams* API, as long as it is a `ReadableStream`.

There are three kinds of *Streams*. The first one is `ReadableStream`. You can think it is source stream that constantly generates data. The second one is `WritableStream`. You can think it is a destination of the data stream. The last one is `TransformStream`, which processes the data from a `ReadableStream` and then write it into a `WritableStream`, or another `TransformStream`, which means you can link as many `TransformStream` as you want.

For *WebCodecs* usage, we can do face recognition in one `TransformStream` and
and background blurring in another, then combine them as our processing pipeline.

### Future of Audio Processing

![audio-processing][audio-processing]

On audio side, we can do similar things to process the `AudioData` with *WebAudio* API.

## Special Thanks & Closing Words

Over the past year, my team has implemented [many modules][fx-webcodecs-July-25-2023] for *WebCodecs*. I need to thank to [Paul Adenot](https://github.com/padenot), [Andrew Osmond](https://github.com/aosmond), media team, and DOM team, to help solving spec issues and engineering problems. Now we alreay have the essential blocks to build the reset of the *WebCodecs*, so I am looking forward seeing what Firefox can bring to the future of media processing on the web!

[webcodecs-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebCodecs_API
[webrtc-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
[webgl-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
[webgpu-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API
[interfaces]: ../images/posts/webcodecs/interfaces.png "WebCodecs Interfaces"
[zoom-hack]: https://webrtchacks.com/zoom-avoids-using-webrtc/
[wasm]: https://developer.mozilla.org/en-US/docs/WebAssembly
[video-processing]: ../images/posts/webcodecs/video_processing.png "Future of Video Processing"
[video-streams]: ../images/posts/webcodecs/video_streams_API.png "Video Processing Pipeline with Streams API"
[streams-api]: https://developer.mozilla.org/en-US/docs/Web/API/Streams_API
[audio-processing]: ../images/posts/webcodecs/audio_processing.png "Future of Audio Processing"
[fx-webcodecs-July-25-2023]: https://github.com/mozilla/gecko-dev/tree/28f4536791dc9f145984ec9004102982ee6cc905/dom/media/webcodecs