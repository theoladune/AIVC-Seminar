# Title slide

Today, i'll be talking about end-to-end learned video coding. I'm going to
introduce the open-source learned video coder that we have designed this year at
orange.

# Intro

So let's start with a brief introduction. Last month, I defended my phD, which
was about the design of learned video coding schemes. In this context, I built
an entirely learning-based video coder, which strives to offer the same features
than a traditional coder. This translates into a learned coder, competitive with HEVC
for different rate targets and able to achieve all the usual coding configuratiuons. 

---

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

So using, the different parameters, we have a quite flexible coder, able to use any coding configuration.

# Setting the rate target

So once we have chosen the coding configuration, we may want to set the rate target.
Currently, one encoder-decoder pair is only able to address a single rate target. 

---

In order to achieve different rates, we provide 7 pre-trained models, which
allows to obtain a rate from a few kilobits per second to a few megabits per second.

--- 

These pre-trained models have the same architecture

---

But their parameters are trained using a different rate constraint

--- 

In practice, you just have to provide which model you want to use to compress the video.

# Competitive performance

The pre-trained models we provide offer performance
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

Moreover, this learned coder scored the best performance among all learned coders at the
CLIC 21 challenge. This further legitimizes the results obtained by our
coder.

# Part 2

Alright, so we've seen how we can use our codec to actually compress videos.
Now, we'll have a look at some of the key elements which are important to obtain
great coding performance.

# System overview

Here, we see an overview of the entire codec. We have a frame to code, and some
reference frames already available at the decoder. The codec has 3 main stages.

---

First, a motion network estimates and transmits motion information, as well as a
coding mode selection. This motion network is built with three neural transforms
: the conditioning, the analysis and the synthesis. I'll detail their role later
on.

---

Then, the motion information is used to compute a temporal prediction through a
motion compensation step. 

---

Finally, once a prediction is available, two coding modes are present to exploit
this prediction. The simplest one is called skip mode, which is a direct copy of
the prediction. The second mode consists in sending a correction to the
prediction using a dedicated neural network. Both coding modes contributions are
added to get the reconstructed frame. The two coding modes are abritrated using
the coding mode selection obtained thanks to the motion network.

# System overview 2

Summing things up, we have a quite usual learned video coder.

---

That is, we have estimation and transmission of motion information. Which is
used to compute a temporal prediction. And finaly, we transmit only the
unpredicted part of the signal.

---

However, when we look more in depth at our proposed coder, it has some
particularities that are responsible for significant performance improvement.

---

First, we have a mecanism to infer some motion information at the decoder-side,
that is, without any transmission.

---

Then, when we want to transmit a correction to the prediction, we use something
more advanced than the usual residual coding.

---

And finally, the presence of an additional coding mode through the skip mode, is
quite uncommon in the learned coding literature even though it helps improving
the coding efficiency.

For the next few slides, i'll give some details on these interesting
particularities of our coder.

# Decoder-side motion

Let's start with the decoder side motion inference. You see here the
architecture of the neural network responsible for the motion information.

---

It has the frame to code and its references as inputs. Two different transforms
take place to extract motion information.

---

The conditioning transform is performed at the decoder and is applied only on the
reference frames. It aims to capture the motion information already available at
the decoder, that is, with no transmission required. This information is shown
as a red cube on the diagram. In practice, it takes the form of some abstract
latent variables.

---

The analysis transform takes place at the encoder and its role is to identify
and transmit only the motion information missing at the decoder. This
information is shown as a green cube on the diagram. In real life, it
corresponds to a second set of latent variables.

---

Finally, both sets of latent variables are combined through the synthesis
transform to obtain the final motion information.

---

Let's have a visual example to better grasp the interest of this mechanism. So
we see here a video on the left and on the right, we have the resulting motion
information, as estimated and sent by our codec. The colour indicates the
direction of the motion while the color intensity describes the magnitude of the
motion.

---

So this is the motion resulting both from the analysis and conditioning transforms.

---

Let's now have a look at the contribution of the conditioning transform alone.
To obtain this visualisation, we just feed the references to the conditioning
transform in order to get the conditioning latent variables. Then we feed only
the conditioning latent variables to the synthesis. This represents what can be
obtained at the decoder, with no transmission. As you can see, the motion of the
people in the background is pretty well infered at the decoder. However, the
girl's motion in the foreground is a bit too complex to be infered at the
decoder, it requires some transmission.

---

This is the role of the analysis transform. We see its contribution here. The
motion of the girl in the foreground is almost entirely sent through the
analysis transform. As expected, the motion in the background, which is already
infered at the decoder is not sent by the analysis.

---

Therefore, the presence of the conditioning transform allows to infer some of
the motion information at the decoder.

---

which allows to spare rate and get more precise motion information.

---

Performance-wise, it's really interesting for the lower rates. On this graph,
you see in red the benefits of motion inference compared to the baseline in
green. While performance remains pretty much the same at high rate, low-rate
performance is significantly improved.

# Exploit the prediction

Ok, so fast-forward in our coding scheme. We've obtained motion information,
we've used it through motion compensation and now we have a prediction
available. In our codec, there are two ways of exploiting this prediction.

---

The simplest one is skip mode, a direct copy of the prediction.

---

The second one consists in sending a correction to the prediction with a neural network.


--- 

These two modes are arbitrated by the coding mode selection coming from the
motion network.


Having these two modes allows the system to select the most suited one
pixel-wise, offering better content adaptation and thus better performance,
reducing the rate by around 10%.

# Sending correction

Let's have a look to the first coding mode where we send a correction to the prediction.

---

To achieve this, we use an architecture identical to the one used for
decoder-side motion inference, which is called conditional coding. 

---

The main idea of conditional coding is to mix in a latent domain the quantities
present at the encoder and the decoder, namely the frame to code and its
prediction. By doing this, we let the system learn both the optimal domain of
representation and the optimal operations to perform the mixture.

---

This results in a richer and more complex mixture than the usual residual coding

---

which offers a rate reduction of 30 percents.

# Skip mode

Besides sending a correction to the prediction, the other coding mode is the skip mode.

---

It copies the areas of the prediction that are well predicted enough and don't require a correction to be sent. 

---

These areas are selected by the motion network.

# Skip mode - visual example

Let us continue the visual example we've considered before to illustrate the
behaviour of the two coding modes. So we have the same video to code. 

---

Thanks to the motion network, we have this motion information. 

---

Actually, as we need to consider b-frames, there are two motion maps computed by
the neural network.

---

These motion maps are used to compute a prediction of the frame to code. We see
here the prediction. It's quite accurate, except for some challenging areas,
such as the ball which has a quite complex motion. As a result, the ball
sometimes disapears from the prediction. It is particularly visible when the
girl makes the ball bounce against the ground, where we can see many artifacts
in the prediction.

---

This prediction is exploited by the two available coding modes. In order to
arbitrate between these two modes, the motion network estimates and
transmits a coding mode selection.

---

Here is this mode selection. The blue-ish areas correspond to the skip mode

---

We can see the skip mode contribution here. As expected, it encompasses all the
areas well predicted enough that can be directly copied. These areas correspond
to the slow-moving objects, such as the background.

---

The red areas in the coding mode are those which require a correction to be
sent. Here are the contribution of the correction neural network. It is mostly
used for the areas for which the prediction is quite inaccurate. The best
example for this is the ball, which is obtained exclusively from this coding
mode.

Ok, so this visual example concludes my talk.

# Conclusion

today, we have presented a codec which is built only
with neural networks. It has some particularities in its design, such as motion
inference at the decoder, conditional coding or the presence of a skip mode. And
that makes it able to get performance competitive with HEVC, for different rate
targets and across the usual coding configurations.

---

This codec is available on github, you can have a look at it if you want to test
its performance!

My thesis manuscript is also freely available, and there are a lot of technical
details about our codec if you want to know a bit more about it.

Thanks for your attention :).