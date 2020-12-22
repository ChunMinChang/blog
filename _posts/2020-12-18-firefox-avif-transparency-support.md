---
layout: post
title: 'Firefox: AVIF Transparency Support'
category:
- Work
tags:
- Firefox
- Media
comments: true
type: photo
imagefeature: "../images/posts/avif-transparency/ff-avif.png"
image: "../images/posts/avif-transparency/ff-avif.png"
date: 2020-12-18 01:34 +0800
---
Transparent AVIF images work in Firefox now!

<!--read more-->

I've finished the AVIF image transparency support in Firefox recently,
in [BMO 1654462][BMO1654462].

The AVIF support can be enabled by toggling the `image.avif.enabled` in `about:config` to `true` (it's `false` by default). As far as I know, it's the last piece of the main AVIF work that has to be done. Hopefully the `image.avif.enabled` would be set to `true` by default soon.

To be honest, I am just lucky enough to pick this topic to implement the last piece.
Most of the hard works are done by our excellent [`mp4parse-rust`](https://github.com/mozilla/mp4parse-rust) developers and the [original AVIF decoder author][BMO1625363]. They give me great advices and help me to achieve this work.

## Before v.s. After

|                  | Before (w/ green background) | After (w/ green background) |
| ---------------- | ------ | ----- |
| I420 w/ I709     | ![I420-BT709-before][I420-BT709-before] | ![I420-BT709-after][I420-BT709-after]
| I444 w/ I709     | ![I444-BT709-before][I444-BT709-before] | ![I444-BT709-after][I444-BT709-after]
| I444 w/ Identity | ![I444-Identity-before][I444-Identity-before] | ![I444-Identity-after][I444-Identity-after]

Test page is [here][avif-transparency-test-page]

## How the AVIF image decoding work

The steps to render an AVIF image to the screen from the raw data are as follows:

1. Extract the AVIF image data from the mp4 container.
   - The AVIF image is stored in the mp4 format, so we need to get its data first
   - we use [`mp4parse-rust`](https://github.com/mozilla/mp4parse-rust/blob/3d9efdc868ce8c5767cea28708fa6512c0ab6d17/mp4parse_capi/src/lib.rs#L1183-L1215) to parse the AVIF image in Firefox
2. Decode the AVIF image to Y-Cb-Cr-Alpha data by [Dav1d or AOM decoder][AVIFDecoder]
   - Dav1d is the video/image decoder for AV1/AVIF format.
   - `libavif` has some examples (or you can use `libavif` directly)
     - [Dav1d][libavif-dav1d-example]
     - [AOM][libavif-aom-example]
3. Convert the Y-Cb-Cr-Alpha data into RGBA data
   - The conversion here is the critical part of this project.
4. Render the decoded RGBA data to the screen
   - This part is relatively easy since Firefox already has APIs to do this job for all other image formats or videos.

## Challenge

The biggest challenge in this project is to convert the Y-Cb-Cr-Alpha data into RGBA data correctly. This conversion is the only part that we don't exactly know how to do in Firefox.

To know how to convert the Y-Cb-Cr-Alpha data into RGBA data, first thing I do is to look at the third-party library used in Firefox to convert the Y-Cb-Cr to RGB data. The third-party library is [*libyuv*][libyuv]. Fortunately, I saw one function that can transform the Y-Cb-Cr-Alpha into RGBA data if the Y-Cb-Cr's data layout is *I420*. There are many different layouts of the Y-Cb-Cr's data, such as *I444* or *I422*. Therefore, the problem is narrowed down to convert the Y-Cb-Cr-Alpha into RGBA data if Y-Cb-Cr's data layout is *not I420*.

The next task is to look at the function that converts Y-Cb-Cr-Alpha to RGBA data for the *I420* layout to see how the conversion is implemented and evaluate if it's possible to apply its manner to other forms.

The answer is yes. The conversion for the *I420* layout is a 3-step process:

1. It converts the Y-Cb-Cr to RGB data and fills the RGB data into the RGBA buffer.
2. It injects the Alpha data into the RGBA buffer.
3. It does the "pre-attenuate" if the caller requests it.

The last two steps are unrelated to the Y-Cb-Cr 's layout. To apply the same approach to the *non-I420* layout, the only thing I need to figure out is how to convert the Y-Cb-Cr to RGB data. The problem now is narrowed down to the conversion for non-I420 layout. Fortunately, the [`libyuv`][libyuv] library already provides such functions. All I need to do is call the operations depends on the Y-Cb-Cr 's layout, then fill the alpha data and do the "pre-attenuate" if required. As a result, the conversion works as follows:

```
YCbCrA_to_RGBA:
  if the layout is I420 format
    call libyuv::I420AlphaToARGB
  else
    convert the Y-Cb-Cr data to R-G-B data via YCbCr_to_RGBA32
    fill the Alpha data to the RGBA32 buffer
    attenuate RGBA data

YCbCr_to_RGBA32:
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
and explore a field I don't have experience in at the end of 2020.

This is my first work in the graphic area, and I enjoy it.
It reminds me that we can always try something new in life.

2020 doesn't stop me from keeping advancing myself.
I am confident that the next year and the following years won't stop me either.
This won't be the last time I try something different.
I look forward to writing a new story in my life!

2021, I am coming!

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

[AVIFDecoder]: https://github.com/mozilla/gecko-dev/blob/bf8688ff888668027347f1c225cdcdd79ab8dca4/image/decoders/nsAVIFDecoder.cpp#L44-L753
[libavif-dav1d-example]: https://github.com/AOMediaCodec/libavif/blob/2a8e22101758494281d50ae33ec76797b354393e/src/codec_dav1d.c#L52-L189
[libavif-aom-example]: https://github.com/AOMediaCodec/libavif/blob/2a8e22101758494281d50ae33ec76797b354393e/src/codec_aom.c#L72-L202
[YCbCrA_to_RGBA]: https://github.com/mozilla/gecko-dev/blob/bf8688ff888668027347f1c225cdcdd79ab8dca4/gfx/ycbcr/YCbCrUtils.cpp#L288-L327