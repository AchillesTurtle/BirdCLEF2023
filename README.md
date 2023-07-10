<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a name="readme-top"></a>




<!-- PROJECT LOGO -->
<br />
<div align="center">

  <h3 align="center">BirdCLEF 2023</h3>

  <p align="center">
    Identify Bird Calls in Soundscapes
  </p>
</div>



<!-- ABOUT THE PROJECT -->
## Overview & Files

This is a write-up of how I tackled the [BirdCLEF2023 challenge](https://www.kaggle.com/competitions/birdclef-2023) on Kaggle, including methodology, failed attempts, and learnings.

All of the code files in this directory were ran in Kaggle and not meant to be run locally.

There is 3 folders included in this folder:
* `Identify_Duplicates`
* `Speed_up_Audio_Processing`
* `Experiments_and_Learnings`
<br>


### Identify_Duplicates

Description of the process of identifying duplicate audio samples to avoid data leakage in the train/validation splits.

### Speed_up_Audio_Processing

Description of why and how can we speed up the audio processing speed on the CPU. This is because Kaggle provides free P100 but only gives us 2-cores CPU, causing a great CPU bottleneck on the preprocessing and data augmentation on audio data.

### Experiments_and_Learnings

Description of the main methods that I tried, reasoning behind them, the results, and also notes on addressing some of the issues.

<br>

## Usage

The code is written and ran on Kaggle, therefore not meant to be run locally.
