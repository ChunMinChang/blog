---
layout: post
title: AVIF transparency support in Firefox
category: [Work]
tags: [Firefox, Media]
comments: true
type: photo
imagefeature: ../images/posts/avif-transparency/ff-avif.png
---

Transparent AVIF images work in Firefox now!

<!--read more-->

I've finished the AVIF image transparency support in Firefox recently,
in [BMO 1654462][BMO1654462].

The AVIF support can be enabled by toggling the `image.avif.enabled` in `about:config` to `true` (it's `false` by default). As far as I know, it's the last piece of the main AVIF work that has to be done. Hopefully the `image.avif.enabled` would be set to `true` by default soon.

To be honest, I am just lucky enough to pick this topic to implement the last piece.
Most of the hard works are done by our excellent [`mp4parse-rust`](https://github.com/mozilla/mp4parse-rust) developers and the [original AVIF decoder author][BMO1625363].
What I did is just to use these APIs to do demux and decode,
then feed the decoded data to the graphics rendering pipeline in Firefox.
## Before v.s. After

|                  | Before (w/ green background) | After (w/ green background) |
| ---------------- | ------ | ----- |
| I420 w/ I709     | ![I420-BT709-before][I420-BT709-before] | ![I420-BT709-after][I420-BT709-after]
| I444 w/ I709     | ![I444-BT709-before][I444-BT709-before] | ![I444-BT709-after][I444-BT709-after]
| I444 w/ Identity | ![I444-Identity-before][I444-Identity-before] | ![I444-Identity-after][I444-Identity-after]

Test page is [here][avif-transparency-test-page]

## How the AVIF image decoding work

1. Demux the AVIF image from the mp4 format
   - The AVIF image is wrapped in mp4 format so we need to demux it first
   - we use [`mp4parse-rust`](https://github.com/mozilla/mp4parse-rust/blob/3d9efdc868ce8c5767cea28708fa6512c0ab6d17/mp4parse_capi/src/lib.rs#L1183-L1215) to parse the AVIF image in Firefox
2. Decode the demuxed image
   - This can done via [Dav1d decoder or AOM decoder][AVIFDecoder]
   - `libavif` has some examples (or you can use `libavif` directly)
     - [Dav1d][libavif-dav1d-example]
     - [AOM][libavif-aom-example]
3. Convert the Y-Cb-Cr-Alpha data into R-G-B-Alpha data
   - The conversion can be done by [`libyuv`][libyuv]'s API (see the detail below)
4. Render the pixels on the screen from the R-G-B-Alpha data
   - TBH, this is a blackbox to me
   - What I did is to feed the R-G-B-Alpha data to graphic pipeline
   - Then image is rendered to the screen

### Challenge

The most challenging part in this work is to convert the Y-Cb-Cr-Alpha data into the R-G-B-Alpha data. Fortunately, the [`libyuv`][libyuv] provides some functions we can reuse. The conversion works as follow:

```
YCbCrA_to_RGBA:
  if the layout is I420 format, then call
  else
    convert the Y-Cb-Cr data to R-G-B data via YCbCr_to_RGB32
    fill the alpha data to the RGB32 buffer

YCbCr_to_RGB32:
  switch layout:
    case I400:
        call libyuv::I400ToARGB
    case I420:
      switch color-space:
        case BT601:
          call libyuv::I420ToARGB
        case BT709:
          call libyuv::H420ToARGB
        case BT2020:
          call libyuv::U420ToARGB
    case I422:
      switch color-space:
        case BT601:
          call libyuv::I422ToARGB
        case BT709:
          call libyuv::H422ToARGB
        case BT2020:
          call libyuv::U422ToARGB
    case I444:
      switch color-space:
        case Identity:
          call GBRPlanarToARGB
        case BT601:
          call libyuv::I444ToARGB
        case BT709:
          call libyuv::H444ToARGB
        case BT2020:
          call libyuv::U444ToARGB
```

To see what the actual code is, please read the code [here][YCbCrA_to_RGBA]

## Closing Word

2020 is a year trapping people in a specific zone and make us feel stuck in life.
I am glad I have an opportunity to embrace a new challenge
and explore a field I don't have experience in, at the end of 2020.

This is my first work in graphic area and I kind of enjoy it.
It reminds me that we can always try something new in life.

2020 doesn't stop me from keeping advancing myself.
I am confident that the next year and the following years won't be able to stop me either.
This won't be the last time I try something different.
I look forward to write a new story in my life!

2021, I am comming!

[BMO1654462]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654462

[BMO1625363]: https://bugzilla.mozilla.org/show_bug.cgi?id=1625363

[BMO1654462-src]: ../images/posts/avif-transparency/BMO1654462-src.png "BMO1654462-src"

[I420-BT709-before]: ../images/posts/avif-transparency/I420-BT709-before.png "I420-BT709-before"
[I420-BT709-after]: ../images/posts/avif-transparency/I420-BT709-after.png "I420-BT709-after"
[I444-BT709-before]: ../images/posts/avif-transparency/I444-BT709-before.png "I444-BT709-before"
[I444-BT709-after]: ../images/posts/avif-transparency/I444-BT709-after.png "I444-BT709-after"
[I444-Identity-before]: ../images/posts/avif-transparency/I444-Identity-before.png "I444-Identity-before"
[I444-Identity-after]: ../images/posts/avif-transparency/I444-Identity-after.png "I444-Identity-after"

[avif-transparency-test-page]: http://chunminchang.github.io/playground/avif/transparency.html

[libyuv]: https://chromium.googlesource.com/libyuv/libyuv/

[AVIFDecoder]: https://hg.mozilla.org/mozilla-central/rev/c1478d03a451#l7.90
[libavif-dav1d-example]: https://github.com/AOMediaCodec/libavif/blob/2a8e22101758494281d50ae33ec76797b354393e/src/codec_dav1d.c#L52-L189
[libavif-aom-example]: https://github.com/AOMediaCodec/libavif/blob/2a8e22101758494281d50ae33ec76797b354393e/src/codec_aom.c#L72-L202
[YCbCrA_to_RGBA]: https://hg.mozilla.org/mozilla-central/rev/c1478d03a451#l3.85