---
layout: post
title: Media Session API
date: 2020-08-08 07:56 +0800
category: [Work]
tags: [Firefox, Media]
comments: true
---

Media Session API will be enabled in Firefox soon!

<!--read more-->

# Media Session API

## What is Media Session API

The media-session API is a set of API to interact with the media-control panel on different platforms.
Those APIs allow developers to customize the media control within a webpage.
The object being controlled (or customized) is named *media-session*.
One *media-session* per page.

In breif, those APIs allow developers to

- customize the media metadata on the platform UI
- get the signals sent from the physical hardware media-key, or the virtual media-key on the platform UI
- set the playback state and position state on the platform UI

## The platform UI interfaces

### On Windows

![SMTC][SMTC]

On Windows, the virtual platform UI shows when the volume keys are pressed.
(Code reference is [here](https://github.com/mozilla/gecko-dev/blob/bf8688ff888668027347f1c225cdcdd79ab8dca4/widget/windows/WindowsSMTCProvider.cpp))

### On Ubuntu (or Other Linux Platforms)

![MPRIS][MPRIS]

On Ubuntu, the virtual platform UI is embedded in the notification bar.
Other Linux platforms may place the virtual platform UI in a similar way.

Since the media-session API is implemented by following the
[*Media Player Remote Interfacing Specification (MPRIS)*][MPRIS-spec],
the third-party application is able to get those media-session information as well.
That means some desktop extensions (e.g., Gnome/KDE extension)
have a way to customize the UI by their needs.
(Code reference is [here](https://github.com/mozilla/gecko-dev/blob/bf8688ff888668027347f1c225cdcdd79ab8dca4/widget/gtk/MPRISServiceHandler.cpp))

### Next/Previous Track Buttons

You may notice that the *prvious track* and the *next track* buttons are disabled in above figures.

By default, only *play*, *pause*, and *stop* are enabled.
the *prvious track* and the *next track* are only enabled
when developers request to use those buttons,
by setting their corresponding button-press handlers.

Youtube set those handlers when the video is put into a list.

![SMTC-YT][SMTC-YT]
![MPRIS-YT][MPRIS-YT]

### Private Browsing

We do respect the users privacy.

When the users use the private browsing window,
no media information will be exposed to the platform UI.

![SMTC-private][SMTC-private]
![MPRIS-private][MPRIS-private]

## Physical Hardware Media Key

The media-session API enables the developers to set the handlers
that are run when users press the media keys.

### Media Keys on Keyboard

![KB-keys][KB-keys]

### Bluetooth Media Controller

![BT-keys][BT-keys]

As long as the button-press handler is set, the handler will be executed
whether the key are from physical hardware or virtual platform UI panel.

The usability of the physical buttons follows the button state of the platform UI.
For example, if the *previous/next track* button on the platform UI is disabled,
the button-press handler for *previous/next track* button won't be executed
when the physical *previous/next track* key (on keyboard or bluetooth controller) is pressed.

## Javascript API

## Set the *Metadata* of the *Media Session*

Set the metadata is easy. See the following example:

```javascript
navigator.mediaSession.metadata = new MediaMetadata({
  title: 'TzChien慈謙 Lantern Light',
  artist: 'TzChien OFFICIAL',
  album: 'Lantern Light EP',
  artwork: [
    {
      src: 'http://tzchien.com/lantetn_light.jpg',
      sizes: '128x128',
      type: 'image/jpg'
    },
  ]
});
```

The platform UI on Windows will be:

![SMTC-panel][SMTC-panel]

The platform UI on Ubuntu will be:

![MPRIS-panel][MPRIS-panel]

You may wonder where the `album` string goes.
Well, the `album` string isn't be shown on those panels.
However, it may be shown on some desktop extension.
Here is a gnome extension I have:

![MPRIS-gnome-extension][MPRIS-gnome-extension]

The above metadata is set by the following code:

```javascript
navigator.mediaSession.metadata = new MediaMetadata({
  title: "The Rabit's Revenge",
  artist: "TzChien慈謙",
  album: "Life as People",
  artwork: [
    {
      src: "http://tzchien.com/life_as_people.jpg",
      sizes: "128x128",
      type: "image/jpg"
    },
    ....
  ]
});
```

## Set the *Action-handler* of the *Media Session*

```javascript
var audio = document.body.querySelector("#player");
audio.src = "My_Blood_Shall_Wake_You_Up.mp3";

navigator.mediaSession.setActionHandler("play", play);
navigator.mediaSession.setActionHandler("pause", pause);
navigator.mediaSession.setActionHandler("previoustrack", previoustrack);
navigator.mediaSession.setActionHandler("nexttrack", nexttrack);
navigator.mediaSession.setActionHandler("seekbackward", seekbackward);
navigator.mediaSession.setActionHandler("seekforward", seekforward);

function play() {
  audio.play();
}

function pause() {
  audio.pause();
}

function previoustrack() {
  audio.src = "The_Rabbits_Revenge.mp3";
}

function nexttrack() {
  audio.src = "Against_the_Time.mp3";
}

function seekbackward() {
  // Seek backward 10 sec.
  audio.currentTime = Math.max(0, audio.currentTime - 10);
}

function seekforward() {
  // Seek forward 10 sec.
  audio.currentTime = Math.min(audio.currentTime + 10, audio.duration);
}
```

The above example is straightforward.
The *media-session* is set to control a *HTML audio element* `audio`.
The current song is `My_Blood_Shall_Wake_You_Up`.

- When the *play* button is pressed, the `audio` is play, or resume.
- When the *pause* button is pressed, the `audio` is paused.
- The `audio` is switched to `The_Rabbits_Revenge` or `Against_the_Time`,
if *previous track* or *next track* button is pressed.
- The playback is sought forward 10 seconds or backforward 10 seconds,
if *seek forward* or *seek backward* button is pressed.

The above code works whether the media key is from physical buttons or virtual buttons on the platform UI.

## Closing Words

The media-session API is a powerful API to customize
not only the playback behavior within a webpage but also the platform UI.

It will be enabled in Firefox in a nearly future.
If you can't wait to try it out, you can play it in the [Firefox Nightly][nightly] now.

For more detail or use cases,
please read the [W3C spec here][w3c-mediasession] or the [MDN here][mdn-mediasession].

If you find any bug, feel free to open a bug on [Bugzilla][BMO].

Stay tune!

[SMTC]: ../images/posts/mediasession/SMTC.PNG "SMTC on Windows"
[MPRIS]: ../images/posts/mediasession/MPRIS.png "MPRIS on Ubuntu"

[SMTC-YT]: ../images/posts/mediasession/SMTC-YT-queue.PNG "Youtube list on Windows"
[MPRIS-YT]: ../images/posts/mediasession/MPRIS-YT-queue.png "Youtube list on Ubuntu"

[SMTC-private]: ../images/posts/mediasession/SMTC-private.PNG "Private browsing on Windows"
[MPRIS-private]: ../images/posts/mediasession/MPRIS-private.png "Private browsing on Ubuntu"

[SMTC-panel]: ../images/posts/mediasession/SMTC-panel.PNG
[MPRIS-panel]: ../images/posts/mediasession/MPRIS-panal.png
[MPRIS-gnome-extension]: ../images/posts/mediasession/MPRIS-gnome-extension.png

[BT-keys]: ../images/posts/mediasession/bt-media-controller.png
[KB-keys]: ../images/posts/mediasession/media-keys-on-keyboard.png

[MPRIS-spec]: https://www.freedesktop.org/wiki/Specifications/mpris-spec/

[w3c-mediasession]: https://w3c.github.io/mediasession/
[mdn-mediasession]: https://developer.mozilla.org/en-US/docs/Web/API/Media_Session_API

[BMO]: https://bugzilla.mozilla.org/home
[nightly]: https://www.mozilla.org/en-US/firefox/channel/desktop/#nightly
