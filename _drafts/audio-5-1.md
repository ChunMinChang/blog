---
layout: post
title: Audio 5.1
category: [Media]
tags: [Firefox]
mathjax: true
comments: true
type: photo
imagefeature: ../images/posts/multichannel/audio5point1.png
---


## Introduction for Audio 5.1
What is Audio 5.1?

## Channel Layout
Do we have particular arrangement for these channels?

### Channels

| Code | Channel Name                     |
| ---- | -------------------------------- |
| M    | Mono                             |
| L    | Left (Front Left)                |
| R    | Right (Front Right)              |
| C    | Center (Front Center)            |
| LS   | Left Surround (Side Left)        |
| RS   | Right Surround (Side Right)      |
| RLS  | Rear Left Surround (Back Left)   |
| RC   | Rear Center (Back Center)        |
| RRS  | Rear Right Surround (Back Right) |
| LFE  | Low Frequency Effects            |

### Layouts

| layout        | Channel Order |
| ------------- | ------------- |
| DUAL-MONO     | L | R |
| DUAL-MONO-LFE | L | R | LFE |
| MONO          | M |
| MONO-LFE      | M | LFE |
| STEREO        | L | R |
| STEREO-LFE    | L | R | LFE |
| 3F            | L | R | C |
| 3F-LFE        | L | R | C | LFE |
| 2F1           | L | R | RC |
| 2F1-LFE       | L | R | LFE | RC |
| 3F1           | L | R | C | RC |
| 3F1-LFE       | L | R | C | LFE | RC |
| 2F2           | L | R | LS | RS |
| 2F2-LFE       | L | R | LFE | LS | RS |
| 3F2           | L | R | C | LS | RS |
| 3F2-LFE       | L | R | C | LFE | LS | RS |
| 3F3R-LFE      | L | R | C | LFE | RC | LS | RS |
| 3F4-LFE       | L | R | C | LFE | RLS | RRS | LS | RS |

## Mixing

We already know that the audio layout can be configured into different type
based on the number of channels.
The question is, what should we do when the input layout from some audio source 
doesn't match some user's output layout?

If the two channel layouts are equal, 
then they must have same numbers of channels and the same order of channels.
(NOTICE: {L, R} and {M, LFE} have different channel numbers.)
Conversely, if two audio settings have different numbers of channels, 
or they have same numbers of channels but different orders(e.g., {L, R} and {R, L}),
then they must have different channel layouts.

When the **input layout is different from the output layout**, 
we need to convert the audio input data to fit the audio output's configuration.
We call it **mixing**.

### Mixing matrix
![Mixing matrix][mixing-matrix]

Although there are various definitions to convert the audio input data 
into the different output data,
they can be summarized into the following equations.
We illustrate their relationships into the above figure.
The value of $$m_{ij}$$ varies from definition to definition.

$$
\begin{align}
L_{out} &= m_{11} \cdot L_{in} +
           m_{12} \cdot R_{in} +
           m_{13} \cdot C_{in} +
           m_{14} \cdot LFE_{in} +
           m_{15} \cdot LS_{in} +
           m_{16} \cdot RS_{in}
\\
R_{out} &= m_{21} \cdot L_{in} +
           m_{22} \cdot R_{in} +
           m_{23} \cdot C_{in} +
           m_{24} \cdot LFE_{in} +
           m_{25} \cdot LS_{in} +
           m_{26} \cdot RS_{in}
\\
C_{out} &= m_{31} \cdot L_{in} +
           m_{32} \cdot R_{in} +
           m_{33} \cdot C_{in} +
           m_{34} \cdot LFE_{in} +
           m_{35} \cdot LS_{in} +
           m_{36} \cdot RS_{in}
\\
LFE_{out} &= m_{41} \cdot L_{in} +
             m_{42} \cdot R_{in} +
             m_{43} \cdot C_{in} +
             m_{44} \cdot LFE_{in} +
             m_{45} \cdot LS_{in} +
             m_{46} \cdot RS_{in}
\\
LS_{out} &= m_{51} \cdot L_{in} +
            m_{52} \cdot R_{in} +
            m_{53} \cdot C_{in} +
            m_{54} \cdot LFE_{in} +
            m_{55} \cdot LS_{in} +
            m_{56} \cdot RS_{in}
\\
RS_{out} &= m_{61} \cdot L_{in} +
            m_{62} \cdot R_{in} +
            m_{63} \cdot C_{in} +
            m_{64} \cdot LFE_{in} +
            m_{65} \cdot LS_{in} +
            m_{66} \cdot RS_{in}
\end{align}
$$

To simplify it, we can rewite this into a matrix form:

$$
\vec{Audio_{out}} =
\begin{bmatrix}
  L_{out} \\ 
  R_{out} \\ 
  C_{out} \\ 
  LFE_{out} \\ 
  LS_{out} \\ 
  RS_{out}
\end{bmatrix}
=
\begin{bmatrix}
  m_{11} & m_{12} & m_{13} & m_{14} & m_{15} & m_{16} \\
  m_{21} & m_{22} & m_{23} & m_{24} & m_{25} & m_{26} \\
  m_{31} & m_{32} & m_{33} & m_{34} & m_{35} & m_{36} \\
  m_{41} & m_{42} & m_{43} & m_{44} & m_{45} & m_{46} \\
  m_{51} & m_{52} & m_{53} & m_{54} & m_{55} & m_{56} \\
  m_{61} & m_{62} & m_{63} & m_{64} & m_{65} & m_{66} \\
\end{bmatrix}
\cdot
\begin{bmatrix}
  L_{in} \\ 
  R_{in} \\
  C_{in} \\
  LFE_{in} \\
  LS_{in} \\
  RS_{in}
\end{bmatrix}
=\vec{Matrix_{mixing}} \cdot \vec{Audio_{in}}
$$

### Downmix

##### Downmix audio 5.1 to stereo(stereophonic sound)
![Downmix 5.1 to stereo][dowmix-5point1-to-stereo]

$$
\begin{align}
L_{out} &= L_{in} +
           \frac{1}{\sqrt{2}} \cdot C_{in} +
           \frac{1}{\sqrt{2}} \cdot LS_{in}
\\
R_{out} &= R_{in} +
           \frac{1}{\sqrt{2}} \cdot C_{in} +
           \frac{1}{\sqrt{2}} \cdot RS_{in} +
\end{align}
$$

##### Downmix audio 5.1 to quad(quadraphonic sound/4.0 surround sound)
![Downmix 5.1 to quad][dowmix-5point1-to-quad]

$$
\begin{align}
L_{out} &= L_{in} + \frac{1}{\sqrt{2}} \cdot C_{in}
\\
R_{out} &= R_{in} + \frac{1}{\sqrt{2}} \cdot C_{in} 
\\
LS_{out} &= LS_{in}
\\
RS_{out} &= RS_{in}
\end{align}
$$

### Upmix

## Implementation
How to implement multi-channels on different platforms?

[mixing-matrix]: ../images/posts/multichannel/mixing-matrix.png "Mixing matrix"
[dowmix-5point1-to-stereo]: ../images/posts/multichannel/dowmix-5point1-to-stereo.png "Downmix 5.1 to stereo"
[dowmix-5point1-to-quad]: ../images/posts/multichannel/dowmix-5point1-to-quad.png "Downmix 5.1 to quad"