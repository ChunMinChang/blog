---
layout: post
title: 'Firefox FAQ: Audio channel layout'
date: 2020-08-20 06:42 +0800
category: [Work]
tags: [Firefox, Media]
comments: true
---

This is a FAQ page for audio channel layout in Firefox.
If you have problem when playing the audio in Firefox with a multi-channel soundcard,
here are some tips for self-debugging.
This page also provides steps to open a Firefox bug with detailed information
that helps our developer to address your issues more efficiently.

<!--read more-->
- [What is *Channel Layout*](#what-is-channel-layout)
  - [How to Setup the *Channel Layout*](#how-to-setup-the-channel-layout)
    - [On Max OS](#on-max-os)
    - [On Windows](#on-windows)
    - [On Linux](#on-linux)
- [Mixing, Downmixing and Upmixing](#mixing-downmixing-and-upmixing)
  - [When will Audio Mixing Happens](#when-will-audio-mixing-happens)
- [Known Driver Issues](#known-driver-issues)
  - [On Mac OS](#on-mac-os)
- [File a Bug](#file-a-bug)
  - [Steps to Create a Bug Report](#steps-to-create-a-bug-report)
  - [Get the Log (for Channel Layout and Mixer)](#get-the-log-for-channel-layout-and-mixer)
    - [On Mac OS](#on-mac-os-1)
    - [On Windows](#on-windows-1)
    - [On Linux](#on-linux-1)

## What is *Channel Layout*

Please see the post [here][audio5point1].

### How to Setup the *Channel Layout*

#### On Max OS

The channel layout can be set via *Audio MIDI setup > Configure Speakers*

![audio-midi-setup][audio-midi-setup]
![configure-speakers][configure-speakers]

#### On Windows

TBC

#### On Linux

TBC

## Mixing, Downmixing and Upmixing

When the **input channel layout is different from the output channel layout**,
we need to convert the audio input data to fit the audio output's configuration.
We call it **mixing**.

The *downmixing* means the number of the input channels is greater than the number of the output channels. On the contrary, the *upmixing* means the number of the output channels is greater than the input channels.

When the the number of the input channels is equal to the the number of the output channels, the channels would just be remapped if the input channels are same as the output channels with different order. Otherwise, the mixing depends on the mixing coefficients from *X* channel to *Y* channel.

To read more detail, please see the [post here][audio5point1]
and the [audio mixer code here][audiomixer].

### When will Audio Mixing Happens

As mentioned above, the mixing will happen when

- either the number of the input channels is different from the number of the output channels
- or the channel layout of the audio source is different from the channel layout of the output hardware
  - e.g. audio source is *[Front-Right, Front-Left]*, while the output hardware is *[Front-Left, Front-Right]*

## Known Driver Issues

### On Mac OS

- Sound Blaster X3 7.1 External USB DAC (see [Bug 1474175](https://bugzilla.mozilla.org/show_bug.cgi?id=1474175#c29))
- Motu 828 MkII (see [Bug 1627827](https://bugzilla.mozilla.org/show_bug.cgi?id=1627827#c15))
- Asus Xonar U7 cannot be set to non-stereo

## File a Bug

If you find the audio output is different from what you expect and you think it's a Firefox bug,
please follow the steps below to open a bug with the following information

It would be very help to upload the below information in to the bug report:
- The *Media* information shown on `about:support` page (see [post here][aboutsupport] for example)
- The program logs

### Steps to Create a Bug Report

1. Open a bug report on [Mozilla Bugzilla](https://bugzilla.mozilla.org/enter_bug.cgi?product=Core&component=Audio%2FVideo%3A+cubeb)
   and set the *Product* and *Compenent* to *Core* and *Audio/Video: cubeb* respectively
   - Please brief describe what's going on, incluing
       1. Steps to reproduce the problem
       2. What's the expected result
       3. What's the actual result
2. Get the log for audio output, by the step-by-step tutorial below
3. Zip all the log files (all the `firefox.*` files below, including `firefox.moz_log` and all `firefox.child-*.moz_log` files. there should be a number in the place of `*`)
4. Upload the zipped file to the opened bug report
5. Paste the information of your `about:support` page to the opened bug report


### Get the Log (for Channel Layout and Mixer)

Here is the command to get the Firefox audio log.
Please attach the log files to the bug report.

#### On Mac OS

1. Open *Terminal* app via *Launchpad* and click *Other*
2. Show what folder you are currently in: Enter `pwd`
3. Enter the command below to launch *Firefox*, which will create log files saved to the folder you are currently in
   - The command format is:
       ```
       MOZ_DISABLE_CONTENT_SANDBOX=1 MOZ_LOG=<MODULE:LEVEL> MOZ_LOG_FILE=<FILENAME> <APP_LOCATION>
       ```
   - For audio problem, please replace `<MODULE:LEVEL>` by
     - `cubeb:4` for audio-setting issues. This should be good for most of the bugs
     - `cubeb:5` for audio sound issues like noise, or glitch
   - `<FILENAME>` can be anyname you like, e.g., `firefox`
   - `<APP_LOCATION>` is
     - For Firefox release: `/Applications/Firefox.app/Contents/MacOS/firefox`
     - For Firefox Nightly: `/Applications/Firefox\ Nightly.app/Contents/MacOS/firefox`
4. Open a tab and play the media/audio/video that (may) play incorrectly on your device and reproduce the problem
5. In your current folder, bundle and zip all the file whose name starts with `<FILENAME>`(e.g., `firefox`). There should be a `"<FILENAME>.moz_log"` and a couple of `"<FILENAME>.child-X.moz_log.0"` (`X` should be an integer)

#### On Windows

1. Search "Command Prompt" application in your Windows 10 machine
2. Launch "Command Prompt" application
3. Type: `set MOZ_LOG=timestamp,rotate:200,sync, <MODULE:LEVEL>`, in the "Command Prompt" application
   - `cubeb:4` for audio-setting issues. This should be good for most of the bugs
   - `cubeb:5` for audio sound issues like noise, or glitch
4. Type: `set MOZ_LOG_FILE=%TEMP%\log.txt`, in the "Command Prompt" application
5. Enter the path to the Firefox executable program to launch Firefox, which also creates the program logs automatically (by above settings)
   - e.g., `"c:\Program Files\Mozilla Firefox\firefox.exe"`,
    `"c:\Program Files (x86)\Mozilla Firefox\firefox.exe"` or`"C:\Users\CM\AppData\Local\Mozilla Firefox\firefox.exe"`
   - The path can be founded by
     1. right click to Firefox app on your desktop
     2. copy the path in the "Target" row
6. Open a tab and play the media/audio/video that (may) play incorrectly on your device and reproduce the problem
7. Go to "C:\Users\<YOUR_USERNAME>\AppData\Local\Temp". The log files should be there
8. Bundle and zip all the file whose name starts with `"log.txt"`. There should be a `"log.txt.moz_log"` and a couple of `"log.txt.child-X.moz_log.0"` (`X` should be an integer)

#### On Linux

TBC

[audio5point1]: audio-5-1#channel-layout
[audiomixer]: https://github.com/ChunMinChang/audio-mixer "An Audio Mixer in Rust"
[aboutsupport]: audio-device-information-on-firefox

[audio-midi-setup]: ../images/posts/multichannel/audio-midi-setup.png "Audio MIDI Setup"
[configure-speakers]: ../images/posts/multichannel/configure-speakers.png "Configure Speakers"
