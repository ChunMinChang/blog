---
layout: post
title: Audio Device Selector
category: [Random]
tags: [Toy]
mathjax: true
comments: true
---
To develop a cross-platform audio library,
I need to switch between computers
with different OS frequently
to check if my code works on different platforms.

<!--read more-->

It's annoying to unconnect the speaker from one computer
and then connect it to another one again and again.
Therefore, I made a device selector that can switch the audio source
of the speaker by simply flipping a switch.

<iframe id="odysee-iframe" style="width:100%; aspect-ratio:16 / 9;" src="https://odysee.com/%24/embed/%40cmchang%3A9%2Faudio-device-switcher%3A3?r=BsRLJWBvcv3MTreGkwZjPR7dTNzfV3Hk&signature=07c4ecc731a92c3304fec8bdb05e7d804ee64d77a9e9ab3e81d9ee834bfdeca36f3dac1dbf6e7ea4f4b51b461c01e291047ea7600d4cd1a83caeee4a0406d3ed&signature_ts=1753743249" allowfullscreen></iframe>


## Circuit

The circuit is super easy. It's nothing more than a switch.
![][deviceselector]

The above circuit diagram is for stereo(*2*-channels) speaker.
If the speaker layout is 5.1(*6*-channels) or 7.1(*8*-channels),
we need to replace the 9-pin switch to 21-pin or 27-pin switch.

In fact, the circuit is just a simple track selector,
so it's not only an audio source selector for a speaker,
but also a speaker switcher for a computer.
That is, you can either share one speaker with two computers,
or share one computer with two speakers.

In general, it needs $$(n + 1) * (k + 1)$$ pins/wires to make a device selector
for one *n*-channels output device with *k* (*n*-channels) input sources,
or *k* output devices with *n*-channels and one (*n*-channels) input source.
The first $$+1$$ is for the *ground wire*,
and the second $$+1$$ is for the *input* or *output* device.

[deviceselector]: ../images/posts/device-selector.png "Device selector"