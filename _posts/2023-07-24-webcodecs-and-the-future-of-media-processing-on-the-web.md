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
type: photo
imagefeature: "../images/posts/webcodecs/icon.png"
image: "../images/posts/webcodecs/icon.png"
---

This post talks about three things: What’s WebCodecs, its demo, and its role in the future of media processing.

<!--read more-->

## What is WebCodces

![goal][goal]

[*WebCodecs*][webcodecs-api] is a set of low-level APIs that can *encode*, *decode*, and *carry* media data **flexibly** and **efficiently**. The flexibility means it can easily work with other APIs such as [*WebRTC*][webrtc-api], [*WebGL*][webgl-api], and [*WebGPU*][webgpu-api]. The efficiency implies transferring data from between WebCodecs and other APIs requires no unnecessary copies.

### WebCodecs Interfaces

![interfaces][interfaces]

*WebCodecs* has nine interfaces: four for audio, four for video, and one for image.

On video side, `VideoFrame` represents the raw video data, and the `EncodedVideoChunk` is its encoded format. `VideoEncoder`, just like its name, is used to encoded a `VideoFrame` into a `EncodedVideoChunk`, while `VideoDecoder` is used to decode a `EncodedVideoChunk` into a `VideoFrame`.

Similarly, *WebCodecs* has symmetric interfaces on the audio side. `AudioData` represents the raw audio data, and the rest interfaces work in the same way as their counterparts.

On top of that, *WebCodecs* has a `ImageDecoder` to decode an image into `VideoFrame`s, so the image processing can follow the pattern of video processing we are going talk later.

On the other hand, it is worth mentioning that `VideoFrame` will be a *processing unit* carrying the image from one interface to another in media processing. The latest [*WebGL*][videoframe-in-webgl] and [*WebGPU*][videoframe-in-webgpu] takes a `VideoFrame` as one of its input sources, and *WebRTC* will have [new API]([insertable-streams]) constantly producing `VideoFrame`s that capture the images from your camera.

### Why We Need WebCodecs

![why-webcodecs][why-webcodecs]

But why we need *WebCodecs*? Right now, in order to achieve low-latency processing, some video conferencing sites like [*Zoom*][zoom-hack], ship their own encoder and decoder to the Firefox. They create their own media encoder and decoder in *C*, then pack them into [*WebAssembly*][wasm], and then send it to the Firefox.

However, without *WebCodecs*, users need to download the encoder and decoder to process the media, which **waste bandwidth**, because they don’t have access to the built-in encoder and decoder in Firefox.

Now, let’s welcome *WebCodecs* to save the day. With *WebCodecs*, users can process media data with the built-in encoder and decoder in Firefox. No *WebAssembly* needed. No duplication of encoder and decoder. And no waste of bandwidth.

## Demo

Now, we are about to see how *WebCodecs* actually work in a video.

![demo-video-options][demo-video-options]

The video will show options of video files in different formats. In this case, we are going to choose h264.

![demo-video-processing][demo-video-processing]

The video will be downloaded, and you will see the original video, and being processed into filters of grayscale, inverted, and sepia.

Let's see the demo (click the images below!).

[![webcodecs-demo](https://img.youtube.com/vi/TMUfP9jXKjc/0.jpg)](https://youtu.be/TMUfP9jXKjc)

## Future of Media Processing

### Future of Video Processing

In the demo, *WebCodecs* decodes the data from the video file, and we adjust the video’s color before painting them to the screen.

![video-processing][video-processing]

Once we integrate *WebCodecs* with *WebGL* and *WebGPU*, or any other graphic APIs, the process can be accelerated by the graphics card, and we will get the images faster.

Again, *WebCodecs* has an interface carrying the image in the video, called `VideoFrame`, that will soon become a processing unit in the *WebRTC* and *WebGL*. As a result, once *WebRTC* can stream the video coming from your camera **in VideoFrame**s, we can easily send them to *WebGL* to process.

![webrtc-video-processing][webrtc-video-processing]

You can put on filters, or blur your video background on your end before sending those data to the internet.
This way, you can have the effects you want on your video, with the process being seamless than ever before.

More specifically, the processing model of the above animation will use the interfaces below. You can follow the green lines to know what interfaces we will use to send the video data from your camera to the internet.

![webcodecs-video][webcodecs-video]

On the sender side, `MediaStreamTrackProcesser` will constantly produce `VideoFrame`s, and the `VideoFrame`s will be processed via *WebGL* or *WebGPU* before being encoded into `EncodedVideoChunk`s by `VideoEncoder`. Finally, those chunks will be sent to the internet and painted to the sender's screen via `<canvas>`.

On receiver side, the process is reversed. The received `EncodedVideoChunk` from the internet will be decoded into `VideoFrame` by `VideoDecoder` and then those frame will be painted to the receiver's screen.

Additionally, the above figure also outlines other usages of `VideoFrame`. It can be created by a `<video>`, a `ImageBitmap`, or other interfaces listed, and it can be an input source of a `<canvas>`, or a `MediaStreamTrack` for *WebRTC*, ...etc.
#### Video Processing Pipeline with Streams API

![video-streams][video-streams]

In fact, we can build a pipeline to process `VideoFrame` stream by [*Streams* API][streams-api].

The `VideoFrame` stream comming from *WebRTC*, or other kind of source, can easily work with *Streams* API, as long as it is a `ReadableStream`.

There are three kinds of *Streams*: `ReadableStream`, `WritableStream`, and `TransformStream`. You can think `ReadableStream` is the source of the data stream, and the `WritableStream` is the destination, while `TransformStream` is a pipe connecting this two for processing the data. `TransformStream` can write the data into a `WritableStream`, or another `TransformStream`, which means you can connect as many `TransformStream` as you want.

Hence, we can do face recognition in one `TransformStream` and
and background blurring in another, then combine them as our processing pipeline.

### Future of Audio Processing

![audio-processing][audio-processing]

On audio side, we can do similar things to process the `AudioData` with [*WebAudio*][webaudio-api] API, but I am going to skip the details right now.

## Wrap Up

![fox-hug][fox-hug]

In the coming days, Firefox is at speed in processing media. Without the waste of power, nor the excessive processing time, we will be able to connect with all the media platforms ever closer.

Special thanks to [Paul Adenot](https://github.com/padenot), [Andrew Osmond](https://github.com/aosmond), Media Team and DOM team, for helping developments. Now we already have the essential blocks to build the rest of the *WebCodecs*, so I am looking forward seeing what Firefox can bring to the future of media processing on the web!

[webcodecs-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebCodecs_API
[webrtc-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
[webgl-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
[webgpu-api]: https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API
[goal]: ../images/posts/webcodecs/webcodecs-goal.png "WebCodecs Goal"
[interfaces]: ../images/posts/webcodecs/interfaces.png "WebCodecs Interfaces"
[videoframe-in-webgl]: https://registry.khronos.org/webgl/specs/latest/1.0/#5.14 "VideoFrame in WebGL"
[videoframe-in-webgpu]: https://gpuweb.github.io/gpuweb/#external-texture-creation "VideoFrame in WebGPU"
[insertable-streams]: https://github.com/w3c/mediacapture-transform "Insertable Streams of Media"
[why-webcodecs]: ../images/posts/webcodecs/why-webcodecs.gif "Why WebCodecs"
[zoom-hack]: https://webrtchacks.com/zoom-avoids-using-webrtc/
[wasm]: https://developer.mozilla.org/en-US/docs/WebAssembly
[demo-video-options]: ../images/posts/webcodecs/webcodecs-video-options.png "Video Options in demo"
[demo-video-processing]: ../images/posts/webcodecs/webcodecs-processing-and-stat.png "Video Processings in demo"
[video-processing]: ../images/posts/webcodecs/webcodecs-video-processing.gif "Future of Video Processing"
[webrtc-video-processing]: ../images/posts/webcodecs/webrtc-webcodecs.gif "Future of Video Processing"
[webcodecs-video]: ../images/posts/webcodecs/webcodecs-video.png "Future of Video Processing Interfaces"
[video-streams]: ../images/posts/webcodecs/streams-api.png "Video Processing Pipeline with Streams API"
[streams-api]: https://developer.mozilla.org/en-US/docs/Web/API/Streams_API
[audio-processing]: ../images/posts/webcodecs/webcodecs-audio.png "Future of Audio Processing"
[webaudio-api]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
[fox-hug]: ../images/posts/webcodecs/fox-hug-media-platforms.png
[fx-webcodecs-July-25-2023]: https://github.com/mozilla/gecko-dev/tree/28f4536791dc9f145984ec9004102982ee6cc905/dom/media/webcodecs