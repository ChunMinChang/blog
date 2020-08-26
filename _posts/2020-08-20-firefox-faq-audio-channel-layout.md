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

## File a Bug

If you find the audio output is different from what you expect and you think it's a Firefox bug,
please follow the steps below to open a bug with the following information

1. The *Media* information shown on `about:support` page (see [post here][aboutsupport] for example)
2. The log for audio output

### Known Driver Issues

#### On Mac OS

- Sound Blaster X3 7.1 External USB DAC (see [Bug 1474175](https://bugzilla.mozilla.org/show_bug.cgi?id=1474175#c29))
- Motu 828 MkII (see [Bug 1627827](https://bugzilla.mozilla.org/show_bug.cgi?id=1627827#c15))
  
### Steps to Create a Bug Report

1. Open the terminal
    - On Mac OS: Open *Terminal* app via *Launchpad* and click *Other*.
      There should be a *Terminal* app inside.
    - On Linux: TBC
    - On Windows: TBC
2. Show what folder you are currently in
    - On Mac OS: Enter `pwd`
    - On Linux: Enter `pwd`
    - On Windows: Enter `cd`
3. Enter the command in next section to launch *Firefox*,
   with log files saved to the folder you are currently in
4. Open a tab and play the media/audio/video that (may) play incorrectly on your device
5. Go to the folder you are currently in (its path is the output of `pwd`)
6. Zip all the `firefox.*` files, including `firefox.moz_log` and all `firefox.child-*.moz_log` files (there should be a number in the place of `*`)
7. Open a bug report on [Mozilla Bugzilla](https://bugzilla.mozilla.org/enter_bug.cgi?product=Core&component=Audio%2FVideo%3A+cubeb)
   and set the *Product* and *Compenent* to *Core* and *Audio/Video: cubeb* respectively
8. Paste the information of your `about:support` page to the opened bug report
9. Upload the zipped file to the opened bug report

### Get the Log for Channel Layout and Mixer

Here is the command to get the Firefox audio log.
Please attach the log files to the bug report.

#### Brief Log

For *Firefox* **release** version:

```sh
MOZ_DISABLE_CONTENT_SANDBOX=1 MOZ_LOG=cubeb:4 MOZ_LOG_FILE=firefox /Applications/Firefox.app/Contents/MacOS/firefox
```

For *Firefox* **nightly** version:

```sh
MOZ_DISABLE_CONTENT_SANDBOX=1 MOZ_LOG=cubeb:4 MOZ_LOG_FILE=firefox /Applications/Firefox\ Nightly.app/Contents/MacOS/firefox
```

The above command will log the audio input and output settings with the channel-layout information.
That would show how Firefox decides to mix or route the audio.

#### Verbose Log

The only difference between the command above and the below
is replacing the `MOZ_LOG=cubeb:4` by `MOZ_LOG=cubeb:5`.
This change will add the audio I/O data in the log files as well.

For *Firefox* **release** version:

```sh
MOZ_DISABLE_CONTENT_SANDBOX=1 MOZ_LOG=cubeb:5 MOZ_LOG_FILE=firefox /Applications/Firefox.app/Contents/MacOS/firefox
```

For *Firefox* **nightly** version:

```sh
MOZ_DISABLE_CONTENT_SANDBOX=1 MOZ_LOG=cubeb:5 MOZ_LOG_FILE=firefox /Applications/Firefox\ Nightly.app/Contents/MacOS/firefox
```

### On Windows

TBC

#### On Linux

TBC

[audio5point1]: audio-5-1#channel-layout
[audiomixer]: https://github.com/ChunMinChang/audio-mixer "An Audio Mixer in Rust"
[aboutsupport]: audio-device-information-on-firefox

[audio-midi-setup]: ../images/posts/multichannel/audio-midi-setup.png "Audio MIDI Setup"
[configure-speakers]: ../images/posts/multichannel/configure-speakers.png "Configure Speakers"
