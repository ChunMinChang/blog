---
layout: post
title: Audio Device Selector
category: [Toy]
tags: [Hardware]
mathjax: true
comments: true
type: video
video: https://www.youtube.com/watch?v=heBQHew3Guc
---
To develop a cross-platform audio library,
I need to switch between computers
with different OS frequently
to check if my code works on different platforms.

It's annoying to unconnect the speaker from one computer
and then connect it to another one again and again.
Therefore, I made a device selector that can switch the audio source
of the speaker by simply flipping a switch.

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/heBQHew3Guc" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe> -->

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