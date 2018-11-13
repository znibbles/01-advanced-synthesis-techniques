## Band Limited Impulse Trains (BLITs)

Let's talk about advanced synthesis techniques. Specifically, the synthesis of classic waveforms that are so straightforward to realize with simple analog circuits, can lead to serious headaches in the digital domain. 

A naive implementation of e.g. a sawtooth will inevitably lead to aliasing, so what are our options?

### Additive Synthesis

is probably the oldest and maybe the most elegant solution, but it’s a little bit counterintuitive as you‘d have to programmatically fade out partials that would otherwise reflect at the Nyquist barrier. 

### Heavy Upsampling

This is a brute force method that will render aliased partials practically inaudible because of reduced amplitude. 

### Wavetable Synthesis

A simple approach that will lead to decent results in many cases. We will look at another one introduced by Stilson and Smith in 200?:

### Bandlimited Impulse Trains

The main idea is pretty simple. We can generate a simple sawtooth by integrating a unit impulse, only that the unit impulse itself contains all frequencies per definition. 

If we bandlimit the impulse, we‘ll have a method to generate a bandlimited sawtooth. 

Let’s approach it from the other side: the impulse response of an ideal (brickwall) anti-aliasing filter is a sinc function. This is the basis of a bandlimited impulse train (BLIT). 



