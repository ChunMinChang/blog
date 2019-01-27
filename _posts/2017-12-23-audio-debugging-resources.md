---
layout: post
title: Audio Debugging Resources
category: [Media]
tags: [Firefox]
comments: true
---

The resources for debugging audio issues.

<!--read more-->

## Software

- Firefox
    - [about:support][aboutsupport]: Show audio device info on Firefox page.
    - [DevTools Media Panel][dtmp-addon] ([source code][dtmp]): An additional panel in the Web Developer Tools displaying technical information about media elements and webrtc.
- FFmpeg: A powerful tool to record, convert and stream audio and video.
  - [website][ffmpeg-website]
  - [source code][ffmpeg-github]
  - [Compilation Guide][ffmpeg-compile]
- [Soundflower][soundflower]: Create virtual (multiple channels) audio device on maxOS.

## Hardware
- [Audio Device Selector][ads]

## Test Samples
- [raw2mp3 (mp3)][raw2mp3]: Convert raw audio files to mp3 files with different header types.
- [SONY: High-Resolution music files(AAC, FLAC)](http://helpguide.sony.net/high-res/sample1/v1/en/index.html)
- [Hyperion Records(MP3, FLAC, M4A)](http://www.hyperion-records.co.uk/testfiles.asp)
- [HiRes(FLAC, including 5.1 Surround)](http://www.2l.no/hires/)
- [Audio Check (wav)](https://www.audiocheck.net/)

## Audio Formats
- [Flac](https://xiph.org/flac/format.html)
  - [FFmpeg](https://github.com/FFmpeg/FFmpeg/blob/49c67e79ca761c43c1310a7e81f8607195a631b9/libavcodec/flac.c)
  - [vlc](https://github.com/videolan/vlc/blob/cc79f1f98f89465385c595f572eee9be1ce80c03/modules/codec/flac.c)

## Code References
- [FFmpeg][ffmpeg-github]
- [vlc](https://github.com/videolan/vlc)

## Others
- [Hydrogenaudio Knowledgebase](http://wiki.hydrogenaud.io/index.php?title=Main_Page)

[aboutsupport]: audio-device-information-on-firefox "Media info on about:support"
[ads]: audio-device-selector "Audio Device Selector"
[raw2mp3]: https://github.com/ChunMinChang/raw2mp3 "Convert raw audio files to mp3 files by various mp3 encoders"

[dtmp-addon]: https://addons.mozilla.org/en-US/firefox/addon/devtools-media-panel/ "DevTools Media Panel"
[dtmp]: https://github.com/mjfroman/media-devtools-panel-react "Media Panel Devtool"

[soundflower]: https://github.com/akhudek/Soundflower "Soundflower"

[ffmpeg-website]: https://www.ffmpeg.org/ "FFmpeg website"
[ffmpeg-github]: https://github.com/FFmpeg/FFmpeg "FFmpeg source code"
[ffmpeg-compile]: https://trac.ffmpeg.org/wiki/CompilationGuide "Compile FFmpeg"