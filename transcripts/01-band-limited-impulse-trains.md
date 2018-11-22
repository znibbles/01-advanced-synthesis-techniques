## Band Limited Impulse Trains (BLITs)

Let's talk about advanced synthesis techniques. Specifically, the synthesis of classic waveforms that are so straightforward to realize with simple analog circuits, can lead to serious headaches in the digital domain. 

Let's do some preliminary setup in this Jupyter Notebook first. You can follow along via [this link](https://colab.research.google.com/github/znibbles/01-advanced-synthesis-techniques/blob/master/01-BLIT.ipynb) if you like. We define a helper method that will simply print a time/frequency plot, which we will use a lot in this episode.

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

The main idea is pretty simple. We can generate a simple sawtooth by integrating a unit impulse minus some DC offset. The negative offset gives us an always falling signal after integration and the impulse corrects this falling signal in regular intervals giving a saw wave. The problem is that the unit impulse itself contains all frequencies per definition, so integrating it will also lead to an infinite number of partials.

Let's demonstrate this by making a simple impulse train. We start the same way as before, just that we set every `P`th sample to 1. To be precise, this is exactly the point where aliasing will have its root: because in almost no case `P` will be integer, we have to assign the unit impulse to the sample that comes _nearest_, by truncating the modulo value:

    y[i] = 1 if np.floor(i % P) == 0 else 0


Let's get the DC component of that impulse train and integrate after removing it. Afterwards, we also remove the DC component of the result. As you can see and hear, there is a fair amount of aliasing present, too.

### Bandlimiting

The idea behind the BLIT is: if we bandlimit the impulse, we‘ll have a method to generate a bandlimited sawtooth. 

Let’s approach it from the other side: the impulse response of an ideal (brickwall) anti-aliasing filter is a sinc function. So if we apply the ideal anti-aliasing filter to the ideal impulse train, we get a bandlimited impulse train (BLIT).

This series of bandlimited impulses can now be sampled without aliasing. I don't want to delve too far into the math here, let's just say that a BLIT is defined by these 3 formulas:

	y(n) = (M/P) Sinc_M[(M/P)n]

where

	Sinc_M(x) = sin(pi*x)/(M*sin(pi*x/M)

and

	M = Number of Harmonics

Calculated via

	M = 2 * floor(P/2)+1

`y(n)` is the sampled BLIT, displayed as a Discrete Summation Formula, where `Sinc_M` is the digital sinc function with `M` harmonics.

Now, we have two edge cases to cover:

- the denominator of `Sinc_M` will be 0 if `n` is an integer factor of `P`
- the numerator of `Sinc_M` will be 0 if `x` is an integer, which will be the case if `M=P`

We have to care for these cases by slightly shifting `P`, otherwise it will lead to instabilities in the generated waveforms.

Let's now make a BLIT. We start the same as before and supply the necessary arguments (`M`), and make an interactive plot. If we integrate the BLIT as above, we get a sawtooth, only this time without aliasing. Neat, isn't it?


### Realtime Capable Sawtooth

To finish this off, let's look at how we can transform this from a static waveform generator into a dynamic oscillator that you can pass a frequency.

First, we have to look at how to integrate in a real-time scenario. Obviously we cannot just take a sum of future samples, so we do this with a **Leaky Integrator**, which is nothing else than a one pole filter with the feedback coefficient alpha very close to 1. That way we keep the filter stable, that's why it's called "leaky". It behaves like a leaky tank that accumulates all water we pour in (intergration) but also has a little leak.

Second, we need a way to subtract the DC component prior to integrating. Because we do not know how the signal will be developing in the future, we take a running average, approximated by a low-pass IIR filter with very slowly decaying impulse response.

If we plot that, we see a bump in the waveform, which is nothing else than the running average slowly kicking in. We can assume that both integrator and moving average filter need to be frequency dependent here, we can try that with another alpha coefficient.

Let's give it a listen. Sounds plausible to me.

## Max Implementation

I did a MaxMSP implementation based on a `gen~` patcher. Inside we have all the components we just discussed:

- a naive saw implementation using just a `[phasor~]`
- a simple impulse train based on a `[click~]` object
- and a BLIT implemented in this `[gen~]` patcher.

The impulse trains are then integrated here with these two `[biquad~]`s. Feel free to reach out if you have any questions on the Max implementation!

## Further Reading

[Stilson & Smith, 1996](https://ccrma.stanford.edu/~stilti/papers/blit.pdf)
[Bandlimited Synthesis](https://www.music.mcgill.ca/~gary/307/week5/bandlimited.html)

## Max Compressed Representation

```
----------begin_max5_patcher----------
2899.3oc2bs0bihiE9YmeErdeoSOoSzE.C6r6C691V0jplGl2ldpTXaYGMMF
X.b5zyTS9suBI.ycDfjytaUckNAAnycc9NRG9iaVsda3qjj0F+Mie1X0p+3l
Uq3WJ6Bqx+6UqO485NeuD9ssNf70vs+556DCkRdMke4DRv92LB7nuPdJw6qE
i6SCH6BOGvuIT9ECNeJ7bpOIk+FA4Wktm+hXu7OgsWe4VoAE2IL+hQdo6dlF
b7oXxtTAsuAdO3NC3FT1+YuI6mXq6AF+R1S7m2bS1OtaYbHaxHLtqClrN+TP
khKk9sHhfDWmPOF34u13W5fgslJCagrxXRjfigPS9egzDK+u+IExq3Iqbcbu
25NCKtN11QGbpv7sO1rKSTzT4BnkPGA3rgksVUX+qePopLvTYVmMb0jPk4f0
mJqeFsCkFZxtY3M.tQG2YS76+OhNCYNUd01EUIjxBUZ6BOchvB62jaez3C+U
im8hOEFP2kbaw3aOucqOoJANtxDL8XlbiRH.WMlooh4vez3CIdmhVDuAAyzP
Mm2br0BqAu0fdJ5reBwHM1iF74.zs0Lam2x8NSdAAyMYKHHzlVbFGiUqW42Y
.mnuHMiq6J3Y2NhngRlw9RvSDesuY6GF6smFdLN7bzDYmt4kRgRjWr2IRJI9
IRfWt4MXxIsw4SgZj+SL2hsXdSImJHnhWcB82IUkdu34etlq0dZRF4rWLCrW
47DaG7CYbQ2hLTuhr6LVu0K3X2wikRzcHL9jGW7XOqLf4gzr.5Y8Ve1jEF+l
ARcqRA2zoyAd.9zlycNN5KYv+9aFf6UHON4..4oJBw7UhM0h5bOwO06MExkV
yLgXWWNSpk7gid1KQ0Vr1yTaJxgz1UG74Ne5tunPk4lYpKyAfi0BStk9am81
m4aB.Vrel8uOAt200RotqH2NYd6ABIsgWvAHRHCrQ5f6+jBUuHmImRhHpKLG
15xxIouzKiOGDvlVCuWHwdGIKAavjMfcMgU0g4Kgp5Dn8Ide4aFrLEIGi8Xq
ktDVbx0HyEH3MGWUvhI6BiHuMLz6cd96JgCfQcvDk0BJMld7HItJKr87ghz8
JeVuyogrDjn6ZHqjxHNunDXfvcECJ+uRmjLxvm7BwmyF2ao9nWv60XnKzjCc
k6WmaSXtvPWmOskD+1voMeHLH8f2tFo0eTHvAiv02wSM2qajVvxH2YyQgsCD
U9VOEtmz.YZaCmrmMfkfN+09OioYyZuxOSayJvtWZQ8dmEeNWcwGFhTn3qaG
uijfdDo3wkVs9sNEbvIWsFAeKRIDWgsq9HhngqJ4VlrklEaJNgFFTYBXQEih
pb4UUdjLQzuFxeQN2UdIZf3RfxKESdg170xduwLBOkQ0miE5yWsKvxjaNDGb
lxeFwEYJqbRhqVxrCRhxMVWuOI5dlxn3ErtRg4DkvQfAvQf2wzpRbYlp+ne3
tuHv0WP1qYq.wRXHJljvVrkspff9KGdO4f2Y+zm51bt93MbopOXm1yqVeLlt
OLHiHpoMxtbwzkkYrX01pLC+NB7h53gY1SLwROClvXxyIa8hyTV4ILfJFLML
zu9PkOmO4PZ9vQzffFRwzvn9Gjsd3yC7raCYCdZn2MejjmNGHF8IlcQ5SIrT
7peed9949u0e8u5EPYK3SRoBU.BTNnn5IOmrKNz2uF+JF4kNFYOyNeG4qz8o
OWKhY1HramFUXDstTKumdjjjV+ZodGSpekjzuID5Utz4s49wOkRNE4y3h52P
s8dspSa0Hc0t9PQ7pG06.808Dla9opi1YFFiT9udSst2.d8tVoioXeYEaoBr
dv9UEA7yiindQxGUtnvtWQARJQARr.PS3TJRTTGDRisg4ger50ag+n+j56cO
0lr8.10Liuw.wVE5TGNh9sGdP41Cly0d.KR8ACAZzd3p5ZfmuqA99MVYU8D9
+KxBzRkEv2K6BQ8SUt.ANeAhXWNDkL15ZuBBiEMvqmbDRv7WwL2WPjRL9pyv
rz0MRnA6dT0l.kP0m6RmX.OhoMtI1otvOUm66EGUeXoZ73cgopGbU8isZb7U
8gwpFNKIwZ0.uEzUbB41XI7lDtSlf5RwtAcIAvKY.eMJ.LIAgM.PL4.iMBfr
QAkMJvrQ.mMN.sQAoIAPMY.qME.aC.ZaTfaCCdaX.bCChaPfb8Alqa.c8.pS
JfccCtqYTjlgmaMdqz42Sxdh6peO8FMtupEJYz5d2e2Zu6d7rGJnd6pD9HIv
Or4auqyuEeUPWQ7+7U+y11lVj8t7BUt9Ui+gAM.98eN9yAOx+cD+2YrNj8mr
k19PD8iud6Ce3wOV7GO73s298qq9BKWis95rKWg1DeVCLZMGoSTZCmGRKsHd
8PpI33JASjX2S3ZASbaPauiRrW0gDCsTIli06l.qUpc0kWz.CzzhmLwXFfAk
cfwkc37y2tnbINsy+8cU3A0pvaoxNQ3x+aTzkgiZDY2.djlK0gLe6FDmXtt.
U0gjod9hYmy0QVbmOuY22vhsjvyw6JrGJVm0n8RZrrZRoAkoZ+ykhBi4pQmM
ABl.Ax4F30lBmJANlp+l1in+BMflbgFJ2.2IWnAj.XDF.0XcFFpdzhs91H7v
kdCv3QEUgZ3r2wBjkMuha4mMHKqqcIp+NCnxq7h0BMRxkFH6q992LjKwLEFl
KUXHN4BH70VXbvOT8ElEhWn3n37eht1hiGzfsAZoBiKGb6qbzTwYUwHhDSC2
y.8Zj0PR9jDiZa5WWMqyhi0BmsPi4Ik0SOfpM0ylqmEzojzPdeaob6nYW9eW
Q2Sx60ol8d3U2J5mNjYHcJQQKJ6tP6DSmNODq52N4u7fAD..p1JwYYFIBww0
eGwZ.BVMNL8JJ.CKJFCvq9yYGN4b1MWldWtL1ubxyZAb8h.nKzVUDCsPY4zD
kUeHr.0QXMnVXpDQ1oQRJpH6.InOpvVVYgqNoBSYkE1BP4U8Ni2K1tTv6Log
6mzf5gzvxRZZ0NFMEcm1nBnrTARiTQgSx3TwFcREVxREsss0rIqzjFWRV2a5
DceTHMHMeQ.DheF7Ph1o0Buo7utlgGjmgb5WVizCoIajqlERTsTgrAofl5jJ
jMHETmK11t9yCWb7trUvuuTFOjwf9kLHFU7KwaDEaqe+R36LCgt1YTHMo0gQ
qtCYHIk4b0W2PVsIreJCzI7hatbk2i1H08Z1Eo4eljde5hzOk0.opqgQmwml
FA25V7M2Q88q2Gey.oPVDOSVTTXeroF+rTnvF+cdLon8SrVVS+pDOX3UsQvy
+3IpYW3jH1DFGNj7YZe8kJ2Qh09gGODS9s4v6Pw2LGLVbDfA4e1v.yT66Seg
b+QO5kdr8Eu3xS1W6Qqy+VSoCbuz8x2k8lS5tKlsG5yrzfBqvXZ0Sy7.suad
axala9vSNybSEqmryL6tm8BBH9kl8kMqs2Kj8O4kxrv1dNkb42RZ1tu7OtUg
GZ2SuWXsSLkAuE+ub7KqLJMfllcVoEeKrpl6XkaJ44v3z90X0t2FeNtpM1Y1
jUdfYMuqKZkxEreZCnSp0OL3XWzwMUy2XVIYP988d6dS5OSjVSMZpYdebCtT
Ddyle8Y4x950eUP5MyhKmAZk8VsJc1Sla3J0kokLR1IxVhIpFf8FITq8IF1d
hgZehqiinQBwydlMkQoZq.kJ1RFdDqhYRFdBtQEyDV1YBtzYBJidxTE7DPVd
BszYRhIREbDxUhIJ6KR0h0RHGYlIUDhDIiMtqJlHjDSjc2Qlf5elqu4IpJnH
RFuMjNVARpYFpG4sTSMTKxaYBto.iYnLovzHbghXQ4lZGcnXgRkcgkJjuRsF
uJhoCkI9mJVhmaviFSuYopYBNlvyUUyznpImVyj.AQi9SNaRZzOxM5C418eb
+8cby9Ml2mwc0ewY3st4Ou4+.VbEAf.
-----------end_max5_patcher-----------
```

