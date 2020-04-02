---
layout: post
title: Living Synthesis Presentation
date: 2020-04-02
categories: presentation
---

* Lots of engineered sound
* Synthesize instrument sounds derived from real audio input

---

### Table of contents:
* Review
* Setting the scene
* Approach 1
* Approach 2
    * Problem 1
    * Problem 2
* And then...?
    * Other notes
* Final remarks

---

# Digital Audio review
* Stored as a set of *discrete* points
* Normalized everything between 0.7 and -0.7 (dBFS where 1.0 and -1.0 is a maximumally saturated signal and 0.0 is no signal)

# Setting the scene
* Picked the violin and flute because respective musical envelopes are already fairly similar
* Used the [Musical Instrument Database by the University of Iowa](http://theremin.music.uiowa.edu/MIS.html) to obtain clean recordings of a flute and violin playing a C5 note for about 1-2 seconds each.
* Take the two recordings and hopefully produce a flute sound synthesized from the violin recording. Ideally, the techique could be applied universally

{% include audio-player.html src="../../../assets/synthesis/fluteC5.mp3" type="audio/mpeg" caption="Flute C5" %}
{% include audio-player.html src="../../../assets/synthesis/violinC5.mp3" type="audio/mpeg" caption="Violin C5" %}

---

# Approach 1

* Terminology:
    * source - the sound we want to modify, i.e. the violin
    * destination - the sound we are comparing the source to and want to make the source sound like, i.e. the flute
    * scaleFactor - to be defined below

* **scaleFactor** array kept track of the ratio between each source and destination sample value



* Used some ratios to figure out which destination sample to compare each source sample to
    * For example, 800 x *R* is where to look for the "800th" sample of the destination if *R* is the ratio between the number of samples in source and destination)


```python
output = [(sample*scaleFactor[i]) for i, sample in enumerate(source)]
```

{% include audio-player.html src="../../../assets/synthesis/first-synthesis.mp3" type="audio/mpeg" caption="The first synthesis" %}

* Actual note had changed

{% include image.html url="../../../assets/synthesis/first-synthesis-analysis.png" height="20rem" description="Top: Synthesized Output, Middle: Flute, Bottom: Violin" %}

* Violin E4 became a very flat A#4\~453Hz

* Could it work universally?

```python
output = [(sample*scaleFactor[math.floor(ratio*i)]) for i, sample in enumerate(source)]
```

{% include image.html url="../../../assets/synthesis/failed-universal-scalefactor.png" height="15rem" description="Not putting the audio file of this one in here to save your ears" %}

* Vibrato?

```python
scale = np.zeros(sourceSamples)
ratio = destSamples/sourceSamples
scale = [0 if source[i] == 0 else dest[math.floor(ratio*i)]/source[i] for i, sample in enumerate(scale)]
```
(^ where the problem comes from)

* Need a new approach entirely

---

# Approach 2

{% include image.html url="../../../assets/synthesis/adsr.jpg" height="15rem" description="Not my diagram, shoutout to lynda.com" %}

* Sample rate/estimated frequency = wavelength - ish
    * 44100/523.25 = 84.2

{% include image.html url="../../../assets/synthesis/vWave.png" height="10rem" description="" %}
{% include image.html url="../../../assets/synthesis/V-zoomed.png" height="10rem" description="Single violin sustain wavelength, let's call it V" %}
{% include image.html url="../../../assets/synthesis/fWave.png" height="10rem" description="" %}
{% include image.html url="../../../assets/synthesis/F-zoomed.png" height="10rem" description="Single flute sustain wavelength, let's call it F" %}

* Both ended up being 84

Two tasks to do:
* The first was how to transform the shape of the violin wave to the flute wave
* The second was how to identify where I needed to apply this transform function on a full violin note

---

### Problem 1: How to transform from the violin wave to the flute wave
* scaleFactor, S, is element-wise division of the flute samples, F and the violin samples V

{% include image.html url="../../../assets/synthesis/S.png" height="10rem" description="scaleFactor, S" %}

* Need scaleFactor, S such that S x V = F but also that if V' was a similar waveform to V, then S x V' is similar to F
    * S can't be too specialized to particular V

* To test S, need to be able to find V' 's within full recording
    * "does my S need to be better or is the V' I'm using too different from V and I need to tighten my search parameters?"

{% include image.html url="../../../assets/synthesis/Vprime.jpg" height="13rem" description="One of the V' 's I was using. Was this too different from my original V or should my S have worked with it?" %}
{% include image.html url="../../../assets/synthesis/VprimeS-fail.png" height="13rem" description="The result of applying my S on the above V', cleary not what I wanted but the scale of the graph skews how bad it really is though" %}

---

### Problem 2: How to figure out where to apply S
* Find subarrays of 84 samples length that were *similar enough* to my original V
    * Misalignment problem

* Arbitrarily picked start of each wavelength
    * Resulting S only made to fit the violin signal only at specific intervals.

{% include image.html url="../../../assets/synthesis/misalignment.jpg" height="13rem" description="Visual representation of the misalignment problem" %}

* How to characterize my V?

* 1 second of sustained a C5 note should get 523 matches

{% include image.html url="../../../assets/synthesis/matching.jpg" height="10rem" description="Why aren't those areas getting matched?" %}
{% include image.html url="../../../assets/synthesis/vWave.png" height="10rem" description="Here is V again for reference" %}

* Variations from wave to wave
    * Missing peak condition

{% include image.html url="../../../assets/synthesis/X.png" height="10rem" description="Here is what that ended up looking like" %}
{% include image.html url="../../../assets/synthesis/idealX.png" height="10rem" description="This is what I think it ideally would have looked like" %}

## And then...?
Warning, it doesn't sound nice

{% include audio-player.html src="../../../assets/synthesis/final.mp3" type="audio/mpeg" caption="The result of approach 2" %}

* Main issue in finding S or in finding V' ?

### Other notes
* Additive method
* Smoothing S
* How to apply to all notes
* Wonder how would have worked for different registers
* Vibrato/ornamentations
* How to apply onto a full musical work

---

[For a full written recap of the project, click here](https://nick-xie.github.io/blog/2020/04/02/Living-Synthesis.html)
