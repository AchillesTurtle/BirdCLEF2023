<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a name="readme-top"></a>




<!-- PROJECT LOGO -->
<br />
<div align="center">

  <h3 align="center">Speed up Audio Processing</h3>

  <p align="center">
    How to overcome the CPU bottleneck
  </p>
</div>


![Not enough CPU](not_enough_cpu.png)
> *What, you didn't hear that? But I think I did.*


## Issue
Although given a good GPU, the bottleneck on Kaggle has been the CPU since it is limited to two cores. In the following I will go through all the things that I found out that can speed up the process and some pitfalls to avoid.
The GPU utilization is around 80% when training on the entire EfficientNetV2 (slightly modified). If freezing the backbone and trained only on the last few layers, it drops down to 20~30%.


>![GPU utilization when trained on full model](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F41b9c9ab540836207676db3b7406070f%2FUntitled.png?generation=1683972254691424&alt=media)
>*GPU utilization when trained on full model.*

![Audio processing stages](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2Fb092ee49871ac9063bc8fd192a13fae9%2FUntitled.png?generation=1683972435356362&alt=media)
There are three stages that the audio goes through before being fed to the model:

- **Load File:** Load the file from the disk to memory
- **Decode Audio**: Audio may be in compressed format and needs to be decompressed before using. For example: .ogg is compressed, .wav is uncompressed.
- **Compute Spectrogram**: Most models requires spectrogram as input.

We'll discuss how to save time in each of these stages.

## 0. Another Route - save as image

I thought this should be mentioned first since this seems to be the fastest approach. 

Use another script to preprocess the entire dataset and save the computed spectrogram as an image. Although in the training process it still needs to be loaded and decoded, because of the small file size it should still be quicker than dealing with audio. 

### Pros:

- Dataset size is small, speeding up the process everywhere
- Can use 4 cpu cores on Kaggle cpu notebook to preprocess

### Cons:

- Limits data augmentation choices
- Quantization to uint8 loses some detail information
- Channels limited to 1,3,4 channels
- Hard to load segmentation of image if only a small area is needed

### Thoughts:

The main concern I have here is that some might mix audio by summing up images. Mixing audio isn’t as simple as summing up the spectrogram so I wouldn’t say it is an ideal augmentation, but if it works then who cares? 

Most of the other issues mentioned above can be resolved if not strictly using an image format. You can use a numpy array instead and scale it to uint8, but the speed difference still needs to be tested.

>![](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F7e0e28bbe89108afdaf8cac7b0d72b93%2FUntitled.jpeg?generation=1683972579292442&alt=media)
>*A spectrogram in 4 channels(CMYK)*

## 1. Load File

**tl;dr:** Presplit audio into segments is the fastest; If not applicable, then use `num_frames` and `frame_offset` arguments.

### Presplit into segments

Often we only need a small segment of the audio but the normal `torchaudio.load()` just loads and decodes the entire file. Write another script to split and save the audio first.

### Pros:

- Up to 10x faster or more depending on audio length

### Cons:

- Slightly limits the variety of data. For example, if cut at every 5secs, you won’t have a sample that looks like the [2.5,7.5] cut.

## Use num_frames and frame_offset

Maybe you need every possible sample in your training data, then `num_frames` and `frame_offset` arguments are extremely helpful since it only loads and decodes a part of the data. 

### Pros:

- Still significantly faster than loading the entire file. Speedup depends on `num_frames` and `frame_offset`.
- No compromise on data

### Cons:

- Still reads all the way up to the frame_offset. So if attempting to read the last 5 secs of the audio, the load time is basically the same as loading the entire file.
>![](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F0c2551105b287f850110e66c0891bd01%2FUntitled.png?generation=1683972645764760&alt=media)
>![](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2Ff4ab16bf4e02a6eac74d00e2d565dd0e%2FUntitled.png?generation=1683972669076437&alt=media)
>*Comparison of how much time needed to load and decode an audio file*

## 2. Decode Audio

**tl;dr:** Convert .ogg to 16-bit .wav files.

I found this to be the most time consuming part. Decoding takes a lot of cpu and unlike video decoding, there is no way to decode audio on the gpu. If we can save it in a decoded state, such as .wav, then it will save tons of time.

## Convert to .wav - is it too large?

The best solution here is to convert to .wav files, but you might have already tried this before and realized that the dataset is too large. The issue here is that the default of `torchaudio.save()` uses 32-bit bitrate which is an overkill since 16-bit is usually sufficient. The bird datasets the officials released on zenodo are also using 16-bit.

```python
torchaudio.save(file_path,sig,sample_rate=sr,bits_per_sample=16,format="wav")
```

The entire dataset is around 44GB which is totally acceptable and I [uploaded it here](https://www.kaggle.com/datasets/lhanhsin/birdclef2023-wav16) so you don’t have to do it again.

![](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F08d373b8149d2f9a17db189dd07e7879%2FUntitled.png?generation=1683972825295270&alt=media)
**What about 8-bit .wav?** Doing so does make the file size even smaller, but it comes with a drawback that it will be prone to noise. If there are noises such as rain sound in the audio, you basically can’t hear anything except huge crinkling noise.

## Preload to memory

Kaggle is quite generous with the RAM, make use of those to get a slight speed up. For example, I preloaded a set of noise files I reuse a lot in a class.

```python
class EnvNoiseLoader:
    def __init__(self, base_path = wandb.config.noise_base_path, noise_p = wandb.config.noise_p):
        self.noise_p = noise_p
        self.noise_list = []
        print("Loading noise files from: {}".format(base_path))
        for dirname, _, filenames in os.walk(base_path):
            for filename in tqdm(filenames):
                if filename.split(".")[-1] in ["wav","mp3","ogg"]:
                    audio_file = torchaudio.load(str(Path(dirname)/Path(filename)),format="wav")
                    self.noise_list.append(audio_file)
  
    def add_random_noise(self,sig,verbose=False):
          # Load noise signal
          noise_sig, noise_sr = random.choice(self.noise_list)
          ... 
          sig = AudioUtil.add_noise(sig, noise_sig)
        
        return sig
```

## Tip - working with large datasets

When datasets are too large it might exceed the allocated disk size of /kaggle/working, however, you can still save up to 100GB in other directories such as /kaggle/tmp and upload it to kaggle directly in the script. A quick tutorial [like this one](https://www.kaggle.com/code/xhlulu/how-to-create-very-large-datasets-from-a-notebook/notebook) should help.

# 3. Compute Spectrogram

Last but not least, computing spectrograms are also very cpu consuming. Fortunately, we can move it to the GPU in both tensorflow and pytorch. In pytorch you can create a mel spectorgram instance easily `torchaudio.transforms.MelSpectrogram().to(device)`

Just remember to send the instance to the GPU!

# 4. Others

- To utilize all CPUs, remember to set the dataloader num_workers argument to 2.
- Set dataloader shuffle argument to False since shuffling the validation set is not needed.