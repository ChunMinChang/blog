---
layout: post
title: Estimation of MP3 Duration
category: [Media]
tags: [Demuxer, Firefox]
mathjax: true
comments: true
---

To calculate the duration of one *MP3* file,
we need to know how it is encoded first.
The estimation of durtation is different
based on how they are encoded.

## CBR(Constant Bitrate) vs VBR(Variable Bitrate) Encoding

*MP3* can be encoded either with **constant bitrate**(**CBR**)
or **variable bitrate**(**VBR**).
The quality of the *MP3* encoded with *VBR* is better than one with *CBR*
since each frame can adopt different bitrate where the music needs it,
while the *CBR* file uses same bitrate regardless of what sound wave is.

## How to know whether the file is CBR or VBR

*MP3* header has two types: **Xing** and **VBRI**.
The *ID* of the *Xing* header is either **Xing** or **Info**;
The *ID* labelled to **VBRI** is *VBRI* header.

The *ID* of *Xing* header marked with **Info**
is definitely encoded with *CBR*.
Nonetheless, assigning *Xing* as the header ID of a *CBR* file
is logically acceptable because *CBR* is a special case of *VBR*.

## How to estimate the duration

### CBR(Constant Bitrate)

The calculation for *CBR* *MP3* is straightforward:

$$
\text{Duration (seconds)} = \frac{ \text{File Size (bits)} }{ \text{Bitrate (bits/second)} }
$$

The unit of *file size* is usually *bytes*,
so the *bit size* of the file is ```file size(bytes) * 8```.

### VBR(Variable Bitrate)

The calculation for *VBR* *MP3* is a little complicated:

$$
\text{Duration (seconds)} = \frac{ \text{ Samples Per Frame $\cdot$ Total Frames (samples)} }{ \text{Sample Rate (samples/second)} }
$$

The duration is **not accurate** when the *total frames* above
is an **estimated** value.
If the total frames isn't predefined, then it could be estimated by:

$$
\text{Estimated Total Frames} = \frac{ \text{File Size} }{ \text{Average Frame Size} }
$$

### Live stream

No matter what type the *MP3* is encoded with,
the duration is calculated by the **file size**,
but what if the *file size* is **unknown**?
Before addressing the question, we should ask
what type of media will have an unknown file size.
The answer is **live stream**.

The live stream can be closed anytime,
hence we don't need to calculate the duration beforehand.
We just need to make sure the position of the playback
stays at the end of the media track and the end-time keeps increasing.

![][live-stream-playback]
(The position stays at the end of the media track during streaming.)

However, for those live streams with **opening remark**,
we still need to estimate how long the opening talks will be
and show the playback UI as they are non-live streams
before they finish introducing and start streaming.
After finishing the opening talks,
the UI should behave as the same as they are live streams.

<iframe width="560" height="315" src="https://www.youtube.com/embed/f53NjLQTafQ" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

(You can check this by opening [*KHNY Honey 103*](http://honey.macchiatomedia.org:8080/stream/1/?lang=en-US%2cen%3bq%3d0.5) directly,
or go to [shoutcast](https://www.shoutcast.com/) and play *KHNY Honey 103* under Genre *Jazz*)


As *file size* is unknown, we should use [*number of frames*](https://www.codeproject.com/Articles/8295/MPEG-Audio-Frame-Header#XINGHeader)
([example](https://searchfox.org/mozilla-central/rev/f6f1731b1b7fec332f86b55fa40e2c9ae67ac39b/dom/media/mp3/MP3FrameParser.cpp#452,475-479))
to calculate the duration of the opening introduction([example](https://searchfox.org/mozilla-central/rev/22c55eb7b7e6494a8615a7af3b613ff899d2cdba/dom/media/mp3/MP3Demuxer.cpp#389-395)).
(The *number of frames* here is same as the *total frames* mentioned
in the estimation of *VBR MP3*'s duration.)

To sum up this case, the ending time shown on playback at first
should be the duration of the opening talk.
After the playback's position reaches the end, it should stay at the end
and the duration should keep increasing during streaming music.

#### How to know whether the file size is unknown

Usually, the *file size* of a live stream will be set to ```-1```.

## Sample code
- Duration: [MP3Demuxer.cpp](https://searchfox.org/mozilla-central/rev/22c55eb7b7e6494a8615a7af3b613ff899d2cdba/dom/media/mp3/MP3Demuxer.cpp#382-420)
- Parsing Header: [MP3FrameParser.cpp](https://searchfox.org/mozilla-central/rev/f6f1731b1b7fec332f86b55fa40e2c9ae67ac39b/dom/media/mp3/MP3FrameParser.cpp#548)

## Useful References

These points are the summary of what I've learned from [bug 1419736][b1419736].
You can see more detail there.
It's my first bug in demuxer field.
I quickly write a note here in case I need to recall it someday.

The following links are some useful resources I found when
I tried to get into this field:
- [MPEG Audio Frame Header](https://www.codeproject.com/Articles/8295/MPEG-Audio-Frame-Header)
- [MPEG Audio Layer I/II/III frame header](http://www.mp3-tech.org/programmer/frame_header.html)
- [MPEG Audio Compression Basics](http://www.mpgedit.org/mpgedit/mpeg_format/mpeghdr.htm)
- [MP3 Info Tag](http://gabriel.mp3-tech.org/mp3infotag.html)
- [How can I get the duration of an MP3 file](https://stackoverflow.com/questions/3505575/how-can-i-get-the-duration-of-an-mp3-file-cbr-or-vbr-with-a-very-small-library)
- [MPEG簡介與如何計算CBR及VBR的播放時間](https://www.crifan.com/files/doc/docbook/mpeg_vbr/release/webhelp/ch04_xing_vbri.html)

[b1419736]: https://bugzilla.mozilla.org/show_bug.cgi?id=1419736 "Bug 1419736 - mp3 time length bug"
[live-stream-playback]: ../images/posts/live-stream-playback.png "live stream playback"