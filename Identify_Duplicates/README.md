<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a name="readme-top"></a>




<!-- PROJECT LOGO -->
<br />
<div align="center">

  <h3 align="center">Identify Duplicates</h3>

  <p align="center">
    Identify Bird Calls in Soundscapes
  </p>
</div>

## Issue
It is not uncommon for users of the xeno-canto bird call database to reupload the same audio file, or sometimes performing slight alterations such cutting and filtering the audio then reupload. The existence of duplicate audio files will cause data leakage when doing train/validation data splits, making the metric too optimistic. The effect is fairly prominent on classes where there is less data, and will have a noticeable change on the padded cmAP metric that we are using.


## Methods

### Initial attempt: Hashing
Assuming that there are duplicate audios, hashing the first 5secs seems to be a fast approach. Suprisingly, none of the audios were had the same hash, even those that are surely identical. A closer look on the data revealed that although the audio and FFT graph hears and looks the same, the values still have a slight difference possibly because of the audio compression/decompression methods.

### Another attempt: Dynamic Time Warping (DTW) distance
A more robust way to compare audio is to use dynamic time warping (DTW) distance as the similarity metric. DTW is an common metric used to measure time series similarity since it is robust to time shift.
<br>
Manual validation is still needed since there exists false positives, for example, being completely silent in the first five seconds. This can be done be observing the spectrogram after subtracting the two signals.

>**True positive**: the subtracted spectrogram is black
![True positive: the subtracted spectrogram is black](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F8282338570c4c34e66dbbfc3dc236093%2F__results___11_80.png?generation=1680065468372086&alt=media)

>**False positive**: the subtracted spectrogram shows parts of respective signals
![False positive: the subtracted spectrogram shows parts of respective signals](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4992459%2F45767965ae6517dbaa1d2fddfff63872%2F__results___11_48.png?generation=1680065526057087&alt=media)

## Conclusion and Thoughts

In the end, there are 18 duplicates discovered. They all seem to be reuploads from the same author, but some of them have differences in call types, ratings, and secondary labels. If those fields are to be used, then maybe it is worth thinking about which row to keep.

### Next steps
DTW is still not the best method for audio file comparison, especially when trimming, filtering, and different bit rates exists for the same file. The best and robust solution here would to run inference through a pretrained model and compare their embeddings, like what *** did here.

