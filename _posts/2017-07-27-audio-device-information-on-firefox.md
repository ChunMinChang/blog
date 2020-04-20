---
layout: post
title: Audio Device Information on Firefox
category: [Work]
tags: [Firefox, Media]
mathjax: true
comments: true
type: photo
imagefeature: ../images/posts/media-on-Firefox-aboutsupport.png
---

The information about the settings for an audio device is necessary
when we are investigating the problems for audio.
Unfortunately, it's hard to know the audio configurations
on the dysfunctional machine.
It usually takes a few days to get this information from the reporters.

## [about:support](about:support)

To make the communication between developers and the issue reporters efficient,
I created a table showing the audio devices information
on **about:support** in *Firefox*.
(You can get more details on [Bug 1197045][b1197045].)

![][aboutsupport]

When reporters file bugs, they can attach these information as well
so we don't need to ask them for their settings.

### Raw Data

![][aboutsupport-rawdata]

The reporters can simply take a screenshot on their *Firefox*,
or click the *Copy raw data to clipboard* button to get something
like the following then paste it to us.

```
...
"media": {
    "currentAudioBackend": "audiounit",
    "currentMaxAudioChannels": 2,
    "currentPreferredSampleRate": 44100,
    "audioOutputDevices": [
      {
        "name": "Internal Speakers",
        "groupId": "AppleHDAEngineOutput:1B,0,1,1:0",
        "vendor": "Apple Inc.",
        "type": 2,
        "state": 2,
        "preferred": 15,
        "supportedFormat": 12336,
        "defaultFormat": 4096,
        "maxChannels": 2,
        "defaultRate": 44100,
        "maxRate": 96000,
        "minRate": 44100,
        "maxLatency": 4272,
        "minLatency": 190
      }
    ],
    "audioInputDevices": [
      {
        "name": "Internal Microphone",
        "groupId": "AppleHDAEngineInput:1B,0,1,0:1",
        "vendor": "Apple Inc.",
        "type": 1,
        "state": 2,
        "preferred": 15,
        "supportedFormat": 12336,
        "defaultFormat": 4096,
        "maxChannels": 2,
        "defaultRate": 44100,
        "maxRate": 96000,
        "minRate": 44100,
        "maxLatency": 4105,
        "minLatency": 23
      }
    ]
  },
  ...
```

### Future Works

I just implemented the basic version.
Some useful information like soundcards and their drivers or versions
are not included yet.
These unsolved tasks couldn't be addressed
without adding new APIs to our underlying audio library.
However, we are currently oxidating our library from *C++* into *Rust*,
so this work is postponed.

The future works can be followed on [Bug 1498731][b1498731].
I might have a chance to work on it again after the library is rewritten.

[aboutsupport]: ../images/posts/media-on-Firefox-aboutsupport.png "Troubleshooting Information"
[aboutsupport-rawdata]: ../images/posts/rawdata-aboutsupport.png "Troubleshooting Information"
[b1197045]: https://bugzilla.mozilla.org/show_bug.cgi?id=1197045 "Bug 1197045 - Add some basic audio hardware/driver/format information to about:support"
[b1498731]: https://bugzilla.mozilla.org/show_bug.cgi?id=1498731 "[Meta] Media section on about:support"
