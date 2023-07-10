<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a name="readme-top"></a>




<!-- PROJECT LOGO -->
<br />
<div align="center">

  <h3 align="center">Experiments and Learnings</h3>

  <p align="center">
    Model for capturing harmonic series and denoising
  </p>
</div>



<!-- ABOUT THE PROJECT -->
## Introduction

In this part I will explain and talk about the ideas behind the two main components of my model. The two ideas are respectively attempting to capturing the harmonic series of bird calls and encouraging the model to preform denoising by using the embedding distance of noisy and clean audio.
<br>
My final ranking in this competition is quite bad due to the lacking training resources, making me unable to do hyperparameter tuning and unfortunately chose a bad learning rate. Ablation tests of whether the ideas help or not is also not ran.


## Methods

### 1. Capturing Harmonic Series

What is harmonic series and why is it relevant? By definition, the note A4 is 440Hz (or 442Hz, or 415Hz if you like vintage) regardless of what instrument you use, but even with the same frequency, human ears can easily distinguish an trumpet A4 from an violin A4! <br>
**Why?**
This is because 440Hz is only the base frequency which defines the pitch, however, the tone and feeling of the sound is shaped by the subtle overtone notes. Those are called the harmonic series!
<br>
For example: If higher notes in the harmonic series are still strong, you will have a sharp feeling. On the other hand, if the higher notes are dim you will have a mellow and soft feeling.

![Harmonic series as musical notation](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e8/Harmonic_series_intervals.png/550px-Harmonic_series_intervals.png)
> *Harmonic series as musical notation with intervals between harmonics labeled. Wikipedia*

![Spectrum of different instruments](https://www.researchgate.net/profile/Pritha-Das-2/publication/263914049/figure/fig2/AS:688495500742657@1541161527587/Spectral-analysis-of-the-select-musical-instruments-The-vertical-axis-is-amplitude-in.ppm)
> *Spectrum from different instruments. Das, Atin & Das, Pritha. (2011). Fractal analysis of different Eastern and Western musical instruments*

Given that bird songs and calls do not always have a fixed melody, it stems the idea to encourage the model to learn more about the timbre.
This is done by aligning the harmonic series within the four input channels.

### 2. Encourage the Model to Denoise

The competition host released some other datasets on zenodo, including the direct labels of when and what frequency range a bird call is in. Given the labels, I can now keep only the bird call in the audio and removing all other sounds, creating a "clean" input. So besides of the training target to classify the bird, I can now add the target to let the embeddings of the original audio and the clean audio be similar.

### Experiments_and_Learnings

TODO
