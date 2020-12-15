---
layout: post
title: AVIF image in Firefox
---

AVIF image will be soon enabled in Firefox!

<!--read more-->

I've finished the last piece of the AVIF image work - transparency support in Firefox recently,
in [BMO 1654462](https://bugzilla.mozilla.org/show_bug.cgi?id=1654462).

The AVIF support can be enabled by toggling the `image.avif.enabled` in `about:config` is to `true` (it's `false` by default). Since it's the last work that has to be done before enabling it on Firefox. The `image.avif.enabled` will soon be set to `true` by default. I am excited to see it since it's my first project in graphic field.

## Before v.s. After


## How the AVIF image decoding work

1. Demux the AVIF image from the mp4 format
   - The AVIF image is wrapped in mp4 format so we need to demux it first
   - we use [`mp4parse-rust`](https://github.com/mozilla/mp4parse-rust/blob/3d9efdc868ce8c5767cea28708fa6512c0ab6d17/mp4parse_capi/src/lib.rs#L1183-L1215) to parse the AVIF image in Firefox
2. Decode the demuxed image binary
   - This can done via [Dav1d decoder or AOM decoder][AVIFDecoder]
   - libavif give some examples
     - [Dav1d][libavif-dav1d-example]
     - [AOM][libavif-aom-example]
3. Convert the Y-Cb-Cr-Alpha data into R-G-B-Alpha data
   - The conversion can be done by libyuv's API
   - See the detail below
4. Render the pixels on the screen from the R-G-B-Alpha data
   - TBH, this is a blackbox to me
   - What I did is to feed the R-G-B-Alpha data to graphic pipeline
   - Then image is rendered to the screen

### Challenge

The most challenging part in this work is to convert the Y-Cb-Cr-Alpha data into the R-G-B-Alpha data.
Fortunately, the libyuv provides some functions we can reuse. The conversion works as follow:

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

[AVIFDecoder]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654462
[libavif-dav1d-example]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654462
[libavif-aom-example]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654462
[YCbCrA_to_RGBA]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654462