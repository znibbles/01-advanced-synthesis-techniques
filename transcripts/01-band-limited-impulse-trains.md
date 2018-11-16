## Band Limited Impulse Trains (BLITs)

Let's talk about advanced synthesis techniques. Specifically, the synthesis of classic waveforms that are so straightforward to realize with simple analog circuits, can lead to serious headaches in the digital domain. 

Let's do some preliminary setup in this Jupyter Notebook first. First, we define a helper method that will simply print a time/frequency plot, which will use a lot in this episode.

Now, a naive implementation of e.g. a sawtooth will inevitably lead to aliasing. Let's look at this first and then draw our conclusions. We implement a `makeNaiveSaw` function which we pass a frequency. As this is just for demonstration purposes, let's make it short, 0.1 seconds, with a sampling frequency of 44100 Hz. Next we calculate the total number of samples and make two numpy ranges (consider them arrays): one for sample indexes, and one for discrete time points.

Now, we compute the sawtooth's period in samples by multiplying the reciprocal of the frequency with the sampling frequency. A naive saw is nothing else than the sample index modulo-divided by the period in samples, normalized to a range of -1 to 1.

Let's make this an interactive plot, with a slider. As we crank up the frequency, the aliased frequencies become more and more apparent in the spectrum. They're even clearly audible. Ok, let's make a 2900 Hz saw, for reference.

So what are our options?

### Additive Synthesis

is probably the oldest and maybe the most elegant solution, but it’s a little bit counterintuitive as you‘d have to programmatically fade out partials that would otherwise reflect at the Nyquist barrier. 

### Heavy Upsampling

This is a brute force method that will render aliased partials practically inaudible because of reduced amplitude. 

### Wavetable Synthesis

A simple approach that will lead to decent results in many cases. We will look at another one introduced by Stilson and Smith in 200?: TODO

### Bandlimited Impulse Trains (BLITs)

The main idea is pretty simple. We can generate a simple sawtooth by integrating a unit impulse, only that the unit impulse itself contains all frequencies per definition. 

If we bandlimit the impulse, we‘ll have a method to generate a bandlimited sawtooth. 

Let’s approach it from the other side: the impulse response of an ideal (brickwall) anti-aliasing filter is a sinc function. This is the basis of a bandlimited impulse train (BLIT). 



