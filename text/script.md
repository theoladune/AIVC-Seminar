# Title slide

Today, i'll be talking about end-to-end learned video coding. I'm going to
introduce the open-source learned video coder that we have designed this year at
orange.

# Intro

So let's start with a brief introduction. Last month, I defended my phD, which
was about the design of learned video coding schemes. In this context, I built
an entirely learning-based video coder, which strives to offer the same features
as a traditional coder. This translates into a learned coder, competitive with HEVC
for different rate targets and able to achieve all the usual coding configuratiuons. 

This talk will have two parts. I'll first introduce the main features of the
codec from a user's point of view. That is, how it is possible to use our codec
to compress videos. Then, i'll focus on a few key elements of our coder, that
are both new and important for the compression performance. 

# Same NN for all frame types

So let's start with the most interesting feature of our codec, which is the
fact that we use a single pair of neural networks to encode and decode any type
of frame. So, we have our neural encoder with its parameters theta encoder
and our neural decoder with its parameters theta decoder. These two neural
networks are trained in such a way that they are able to compress a frame $x_t$
while exploiting information from zero to two reference frames.

Here you see the coding of an I-frame, with no reference frame. 

---

But the same neural networks, with the same parameters can also compress P
frames, that is they can exploit one reference frame. 

--- 

And they're also able to compress B frames by using two reference frames.

---

So, we have a single pair of neural networks to code I, P and B frames. This
allows us to arrange these frame types as desired to achieve any coding
configuration, with any intra frame period and gop size.

# Setting the coding configuration

Let's have a look at how we can set the desired coding configuration with our
codec. These are real lines of code that allows to compress videos using our codec. 

---

The first four lines indicates the input raw yuv video, the compressed bitstream
and the decoded yuv video.

---

Then we set the desired coding configuration, here RA for random access. We also
use an intra period of 16 and a gop size of 16. The corresponding coding
configuration is depicted below.

--- 

It is possible to change the GOP size form 16 to 8

---

Or the intra period

--- 

Or the coding configuration, from random access to low-delay P which does not
rely on B frames

---

And we can even do all intra coding with only standalone images.

# Setting the rate target

So once we have chosen the coding configuration, we may want to set the rate target.
Currently, one encoder-decoder pair is only able to address a single rate target. 

---

In order to achieve different rates, we provide 7 pre-trained models, which
allows to obtain a rate from a few kilobits per second to a few megabits per second.

--- 

These pre-trained models have the exact same architecture

---

But their parameters are trained using a different rate constraint

--- 

In practice, you just have to provide which model you want to use to compress the video.

# Competitive performance

The pre-trained models we provide alongside the code offer performance
competitive with HEVC. On this graph, you see rate-quality curves, for 3 coding
configurations : all intra, low-delay P and random access. The dashed lines
represent the performance of HEVC, evaluated through the HM. The solid lines
represent the performance of our coder. The dataset used for these experiments
is from the clic challenge on learned video coding, which was held at CVPR this
year.

Roughly speaking, we see that the learned codec is competitive with HEVC for
random access and low-delay P, and it is significantly better for all intra
coding.

---

To sum up, with this open source coder, we provide a tool to compress videos
using a learned coder which has performance on par with HEVC for different
coding configurations and rate targets.

---

This learned coder scored the best performance among all learned coders at the
CLIC 21 challenge. This further legitimizes the results obtained by our
coder.