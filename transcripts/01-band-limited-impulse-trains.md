## Band Limited Impulse Trains (BLITs)

Let's talk about advanced synthesis techniques. Specifically, the synthesis of classic waveforms that are so straightforward to realize with simple analog circuits, can lead to serious headaches in the digital domain. 

Let's do some preliminary setup in this Jupyter Notebook first. We define a helper method that will simply print a time/frequency plot, which we will use a lot in this episode.

Now, a naive implementation of e.g. a sawtooth will inevitably lead to aliasing. Let's look at this first and then draw our conclusions. We implement a `makeNaiveSaw` function which we pass a frequency. As this is just for demonstration purposes, let's make it short, 0.3 seconds, with a sampling frequency of 44100 Hz. Next we calculate the total number of samples and make two numpy ranges (consider them arrays): one for sample indexes, and one for discrete time points.

Now, we compute the sawtooth's period in samples by multiplying the reciprocal of the frequency with the sampling frequency. A naive saw is nothing else than the sample index modulo-divided by the period in samples, normalized to a range of -1 to 1.

Let's make this an interactive plot, with a slider. As we crank up the frequency, the aliased frequencies become more and more apparent in the spectrum. They're even clearly audible.

So what are our options?

### Additive Synthesis

is probably the oldest and maybe the most elegant solution, but it’s a little bit counterintuitive as you‘d have to programmatically fade out partials that would otherwise reflect at the Nyquist barrier. 

### Heavy Upsampling

This is a brute force method that will render aliased partials practically inaudible because of reduced amplitude. 

### Wavetable Synthesis

A simple approach that will lead to decent results in many cases. We will look at another one introduced by [Stilson and Smith in 1996](https://ccrma.stanford.edu/~stilti/papers/blit.pdf):

## Bandlimited Impulse Trains (BLITs)

The main idea is pretty simple. We can generate a simple sawtooth by integrating a unit impulse, only that the unit impulse itself contains all frequencies per definition, so integrating it will also lead to an infinite number of partials.

Let's demonstrate this by making a simple impulse train. We start the same way as before, just that we set every `P`th sample to 1. To be precise, this is exactly the point where aliasing will have its root: because in almost no case `P` will be integer, we have to assign the unit impulse to the sample that comes _nearest_, by truncating the modulo value:

    y[i] = 1 if np.floor(i % P) == 0 else 0


Let's get the DC component of that impulse train and integrate after removing it. Afterwards, we also remove the DC component of the result. As you can see and hear, there is a fair amount of aliasing present, too.

### Bandlimiting

The idea behind the BLIT is: if we bandlimit the impulse, we‘ll have a method to generate a bandlimited sawtooth. 

Let’s approach it from the other side: the impulse response of an ideal (brickwall) anti-aliasing filter is a sinc function. So if we apply the ideal anti-aliasing filter to the ideal impulse train, we get a bandlimited impulse train (BLIT).

This series of bandlimited impulses can now be sampled without aliasing. I don't want to delve too far into the math here, let's just say that a BLIT is defined by these 3 formulas:

$$ y(n) = (M/P) Sinc_M[(M/P)n] $$
where


$$ Sinc_M(x) = \frac{sin( \pi x)}{M sin(\pi x /M)}$$

$M$ = Number of Harmonics

Calculated via
$$ M = 2\cdot floor(P/2)+1 $$

$y(n)$ is the sampled BLIT, displayed as a Discrete Summation Formula, where $Sinc_M$ is the digital sinc function with $M$ harmonics.

Now, we have two edge cases to cover:

- the denominator of $Sinc_M$ will be 0 if $n$ is an integer factor of $P$
- the numerator of $Sinc_M$ will be 0 if $x$ is an integer, which will be the case if $M=P$

We have to care for these cases by slightly shifting $P$, otherwise it will lead to instabilities in the generated waveforms.

Let's now make a BLIT. We start the same as before and supply the necessary arguments ($M$), and make an interactive plot. If we integrate the BLIT as above, we get a sawtooth, only this time without aliasing. Neat, isn't it?


