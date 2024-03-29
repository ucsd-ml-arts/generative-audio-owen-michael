# Project 2: Generative Audio

Owen Jow, owen@eng.ucsd.edu

## Abstract

In this project, I execute a musical "style transfer" from arbitrary audio to the same audio performed by chickens. I follow the approach of [A Universal Music Translation Network](https://arxiv.org/pdf/1805.07848.pdf) by Mor et al., which trains and uses a single WaveNet encoder in tandem with multiple WaveNet decoders, with one decoder for each "style" of audio. The encoder takes raw audio and embeds it in a latent space, and each decoder takes the latent embedding and recreates the audio in the timbre of its training background. The authors provide code for their paper [here](https://github.com/facebookresearch/music-translation), which I have of course hijacked.

I freeze the published pre-trained encoder (and domain confusion network) and train only a decoder on chicken sounds, on the basis that I am not aiming to translate _from_ the chicken domain, only _to_. The chicken sounds are obtained from random YouTube videos; as a preprocessing step, I split and remove silences from the audio. Originally I had planned to use Google's AudioSet dataset, which indicates YouTube clips containing sounds of different categories (one of which being "[chickens](https://research.google.com/audioset/dataset/chicken_rooster.html)/[roosters](https://research.google.com/audioset/ontology/chicken_rooster.html)"),
but I found these clips to be too full of other, non-chicken utterances (e.g. humans talking).

> _emulate the sounds of chickens, composing them in a way that is at least somewhat melodic to the human ear_

## Future/Alternative Directions

- Based on approaches from [NSynth](https://arxiv.org/pdf/1704.01279.pdf), [GANSynth](https://arxiv.org/pdf/1902.08710.pdf), and [SynthNet](https://www.ijcai.org/proceedings/2019/0467.pdf), I could train a synthesizer to produce chicken timbre conditioned on MIDI sequences. In the former cases (NSynth, GANSynth), I would condition the decoder/generator on a pitch vector in addition to the standard latent vector. For inference, I would encode chicken audio, fix the latent conditioning vector, and then feed in pitch vectors according to a MIDI sequence. In the latter case (SynthNet), I would directly train a WaveNet-like architecture to map (MIDI, style label) to waveform. However, if I wanted to train any of these models on chicken audio (perhaps not necessary for NSynth/GANSynth), I would need pitch- or MIDI-annotated squawk waveforms. I might be able to create those annotations using a library such as [`aubio`](https://aubio.org).
- [This paper](https://arxiv.org/pdf/1910.06711.pdf) (MelGAN: Generative Adversarial Networks for Conditional Waveform Synthesis, with code [here](https://github.com/descriptinc/melgan-neurips)), linked by Robert, might be useful. I could follow their approach from Section 3.3 and replace the WaveNet decoder in the translation network with a MelGAN generator. This would make translation incredibly fast in comparison to what I'm currently dealing with. However, somewhat ironically I lack the time to train the model. (The authors mention they train each decoder for four days on a 2080 Ti on MusicNet data, whereas I have less time, worse hardware, and lower-quality data.)
- [This recent paper](https://arxiv.org/pdf/1811.09620.pdf) (TimbreTron) might yield superior results, but I don't have time to implement it.
- For this project, I could of course experiment with other domain translations. There exists a vast space of music and music genres that might be translated into chickensong. But I don't have a MelGAN, so translation is super slow.
- I expect that my results can be improved with more data and more training.

## Model/Data

- You can download the trained model from [Google Drive](https://drive.google.com/file/d/1p8LoXG6CY5FsNFxf4mUFlz7mtZ3CDDJZ/view?usp=sharing). Place it in the `chickensong/music-translation/checkpoints` directory. (The resulting folder hierarchy will be `chickensong/music-translation/checkpoints/chickenNet`.)
- To download and preprocess the data, run `./make_dataset.sh <desired data root>`.
- To download the outdated AudioSet data, get the [unbalanced train split](https://research.google.com/audioset/download.html) and run
```
python3 dl_train_segments.py unbalanced_train_segments.csv --out_dir <raw wav dir>
for fpath in <raw wav dir>/*.wav; do python3 remove_silences.py ${fpath} --overwrite; done
./preprocess_data.sh <raw wav dir> <processed wav dir>
```

## Code

The training code is in the [`music-translation`](https://github.com/chickensong/music-translation) submodule.

To generate translated audio, first follow the `music-translation` [setup instructions](https://github.com/chickensong/music-translation#setup). Make sure to download the pre-trained models ([direct link](https://dl.fbaipublicfiles.com/music-translation/pretrained_musicnet.zip)) and place them in the `chickensong/music-translation/checkpoints` directory. (The resulting folder hierarchy will be `chickensong/music-translation/checkpoints/pretrained_musicnet`.) Then run
```
cd music-translation
./train_decoder.sh <data root>  # OR download pre-trained model
./sample_chickens.sh <preprocessed folder>  # warning: slow as hell
```
An example of a `<preprocessed folder>` is `musicnet/preprocessed/Bach_Solo_Cello`.

## Quickstart

```
git clone --recursive https://github.com/ohjay/chickensong.git
cd chickensong
```
Follow setup instructions for the `music-translation` submodule, including downloading the pre-trained models (see [Code](https://github.com/ohjay/chickensong#code)). Then, depending on whether or not you are using the pre-trained chicken model, you have two options.

### If using the pre-trained chicken model
Download the chicken model (see [Model/Data](https://github.com/ohjay/chickensong#modeldata)). Then run
```
cd music-translation
./sample_chickens.sh <preprocessed folder>
```

### If training the model yourself
```
./make_dataset.sh .
cd music-translation
./train_decoder.sh ..
./sample_chickens.sh <preprocessed folder>
```

## Results

I use [MusicNet](https://homes.cs.washington.edu/~thickstn/musicnet.html) as a source of audio to be translated into chicken. (Note that the pre-trained `music-translation` models were trained using MusicNet, so there is some precedent.) My chicken model was trained for 200 epochs on about 15 total minutes of unique audio, collected from a hand-curated set of six YouTube videos (see [`make_dataset.sh`](https://github.com/ohjay/chickensong/blob/master/make_dataset.sh) for links).

- [**`cambini_wind_ft_chicken.wav`**](https://drive.google.com/file/d/1baf4mkBw-xL56-2pB9BOdmGRPXTQtJ45/view?usp=sharing): chicken rendition of a [Cambini wind quintet](https://drive.google.com/file/d/1KZPKWLSAtANjqyqZmsshKPDkG_3k5BRw/view?usp=sharing).
- [**`bach_cello_ft_chicken.wav`**](https://drive.google.com/file/d/1eqmXrrtqt2meE1NTdRAX3LDSyGf_ttmS/view?usp=sharing): chicken rendition of a [Bach cello solo](https://drive.google.com/file/d/1IFbcnpKFjkJJ87Hdld5LSB1Q2saElbb9/view?usp=sharing).

### From chicken to classical music

The first direction didn't work very well, so I tried going the other way around: encoding the chicken audio and decoding it using some of the pre-trained instrument decoders. This can be seen as the classical-musical perception of the tunes the chickens are "trying to sing."

- [**`beethoven_from_chicken.wav`**](https://drive.google.com/file/d/1ca4BJ5Id0F09ObEGCQEIop34MXw72R3f/view?usp=sharing): from [chicken](https://www.youtube.com/watch?v=IpNgah-e6v4) ([WAV](https://drive.google.com/file/d/1xPADH_D3cIqdZAPtX1JUSN1YjHsX-CAi/view?usp=sharing)) to a Beethoven string quartet.
- [**`violin_from_chicken.wav`**](https://drive.google.com/file/d/11phXCaam2sqDmK99X1sHD78k1P10nPFe/view?usp=sharing): from chicken ([WAV](https://drive.google.com/file/d/1DC2FlJIdmMOEk_MLlAX1pqEy1m5uSNWg/view?usp=sharing)) to a Beethoven violin.

### Autoencoder reconstructions

I include reconstructions of training and validation data given by the denoising WaveNet autoencoder.

- [`train_input.wav`](https://drive.google.com/file/d/1MpWNYzBp0Pnx_fKSM-CBy-Nd4WM9NiMb/view?usp=sharing) --> [**`train_reconstruction.wav`**](https://drive.google.com/file/d/13HJznavpoNeJO3upARbL8j2_NzIrufya/view?usp=sharing)
- [`validation_input.wav`](https://drive.google.com/file/d/1fzG0RqO2KEpjjLaxFmYONZv5c-cqEBq3/view?usp=sharing) --> [**`validation_reconstruction.wav`**](https://drive.google.com/file/d/1FMDvvxaxE86J5oy4jMPgzYN6VSe_-sVJ/view?usp=sharing)

### Bonus: Chickenspeak

I already had the [code for it](https://github.com/ohjay/visual-questioner/blob/master/tts.py) (note: as a cleaned-up fragment of [CorentinJ's excellent voice cloning project](https://github.com/CorentinJ/Real-Time-Voice-Cloning)), so I figured I'd try performing voice cloning on a chicken. This was the result: [**`chickenspeak.wav`**](https://drive.google.com/file/d/14XQlXCi2IB_jrGRVVVKD8gVfMRqS47qC/view?usp=sharing). The input text was "kloek kloek kakara-kakara	kotek-kotek kokoda guaguagua petok kudak kackel po-kok kuckeliku kokarakkoo kukuruyuk kukeleku quiquiriquic kikiriki" (a collection of [onomatopoeias](https://en.wikipedia.org/wiki/Cross-linguistic_onomatopoeias#Animal_sounds) for chickens clucking and crowing). The "voice" was cloned from [this video](https://www.youtube.com/watch?v=Y2qWdZzSKkw).

[Another example.](https://drive.google.com/file/d/1jNp1IaQwvg7IEzsByd9HXte-pfp-w-nA/view?usp=sharing) / [Speaking actual English (?).](https://drive.google.com/file/d/1xj9Dfk0EgWN8qCKRNvsgacsphm5SNg5n/view?usp=sharing)

## Technical Notes

- The code runs on Ubuntu 18.04 with Python 3.6.8.
- To get the data, you'll need `youtube-dl`. You can install it with pip.
- Other than that, the requirements are PyTorch, librosa, SciPy, tqdm, etc. Nothing too unusual.

## Reflections

As you can hear, this project ended up being more challenging than I had anticipated and the results were not stellar. I attribute this mainly to the fact that my chicken data was not very structured (unlike classical music), and it's inherently a difficult task to translate irregular clucking and background noise into fluid music using current audio-based domain translation methods. This is evidenced by the fact that the autoencoder reconstructions are chicken-like, but the translations are less obviously so. There are quite a few semantic facets of audio to get right in a domain translation: timbre, rhythm, melody, "foreground sound" in the case of my messy chicken data, volume, etc. So perhaps some kind of forced disentanglement (e.g. by conditioning on different aspects of the audio, or by generating each component separately before combining them) would be helpful, to exploit the structure of audio in a learning context.

## References

- Papers
  - [A Universal Music Translation Network](https://arxiv.org/pdf/1805.07848.pdf)
- Repositories
  - [`music-translation`](https://github.com/facebookresearch/music-translation)
  - [`Real-Time-Voice-Cloning`](https://github.com/CorentinJ/Real-Time-Voice-Cloning)
- Datasets
  - [MusicNet](https://homes.cs.washington.edu/~thickstn/musicnet.html)
  - [AudioSet](https://research.google.com/audioset)
