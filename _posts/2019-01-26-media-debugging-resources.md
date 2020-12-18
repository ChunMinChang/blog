---
layout: post
title: Media Debugging Resources
category:
- Work
tags:
- Firefox
- Media
comments: true
---
The resources collection for debugging audio/video issues.

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
- [REAPER][reaper]: Digital audio workstation for recording, mixing, ..., etc.

## Hardware

- [Audio Device Selector][ads]: Share one output/input deivces with multiple input/output devices.

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
- [rhythmbox (Ubuntu's default music player)](https://github.com/GNOME/rhythmbox)
- [clementine-player](https://github.com/clementine-player/Clementine)

## MPRIS

- [spec](https://specifications.freedesktop.org/mpris-spec/2.2/)
- [rhythmbox (via *Gnome*'s `GDBusConnection`)](https://github.com/GNOME/rhythmbox/blob/353c9095c08e12110d94e5fc25a67ce4929e29e0/plugins/mpris/rb-mpris-plugin.c)
- [clementine-player (via *Qt*'s `QDBusMessage`)](https://github.com/clementine-player/Clementine/blob/b007e54b3db9a8f3bc6b49930d7d7b080789c286/src/core/mpris2.cpp)
  - [doc](https://github.com/clementine-player/Clementine/wiki/Controlling-Clementine-from-the-commandline-with-DBus-and-MPRIS)
- [media-explorer](https://github.com/media-explorer/media-explorer/blob/master/plugins/mpris/mex-mpris-plugin.c)
- [Mopidy-MPRIS](https://github.com/mopidy/mopidy-mpris)
- [mpris-controller](https://github.com/gsavin/mpris-controller)
- [doc from  xmms2](https://github.com/xmms2/wiki/wiki/MPRIS)

## AVIF

- [libavif](https://github.com/AOMediaCodec/libavif)
- [cavif-rs](https://github.com/kornelski/cavif-rs)
- [colorist](https://github.com/joedrago/colorist)
- [vantage](https://github.com/joedrago/vantage)

### Utils

- [Scripts to call methods, query properties](https://gist.github.com/ChunMinChang/06a776185055d03ec52cdc1539fd3c23)
  - [`dbus-send` command](https://dbus.freedesktop.org/doc/dbus-send.1.html)
- [mpris-rs](https://github.com/Mange/mpris-rs)
  - Show properties: `$ cargo run --example capabilities`

## Others

- [Hydrogenaudio Knowledgebase](http://wiki.hydrogenaud.io/index.php?title=Main_Page)

## Frequently Used Script

### FFmpeg

Show metadata:

```sh
ffmpeg -i <filename>
```

Show packet data:

```sh
ffprobe -show_packets <filename>
```

Create a sine wav file:

```sh
ffmpeg -f lavfi -i "sine=frequency=441:duration=5" sine_441hz_5s.wav
```

### OSX

#### Unload/Reload the devices

For the system devices:

```sh
sudo kextunload /System/Library/Extensions/AppleHDA.kext
sudo kextload /System/Library/Extensions/AppleHDA.kext
```

For the Soundflower:

```sh
sudo kextunload -b com.Cycling74.driver.Soundflower
sudo kextload -b com.Cycling74.driver.Soundflower
```

[aboutsupport]: audio-device-information-on-firefox "Media info on about:support"
[ads]: audio-device-selector "Audio Device Selector"
[raw2mp3]: https://github.com/ChunMinChang/raw2mp3 "Convert raw audio files to mp3 files by various mp3 encoders"

[dtmp-addon]: https://addons.mozilla.org/en-US/firefox/addon/devtools-media-panel/ "DevTools Media Panel"
[dtmp]: https://github.com/mjfroman/media-devtools-panel-react "Media Panel Devtool"

[soundflower]: https://github.com/akhudek/Soundflower "Soundflower"
[reaper]: https://www.reaper.fm/

[ffmpeg-website]: https://www.ffmpeg.org/ "FFmpeg website"
[ffmpeg-github]: https://github.com/FFmpeg/FFmpeg "FFmpeg source code"
[ffmpeg-compile]: https://trac.ffmpeg.org/wiki/CompilationGuide "Compile FFmpeg"
