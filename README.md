[![test](https://github.com/k2kobayashi/crank/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/k2kobayashi/crank/actions/workflows/ci.yml)
[![PyPI version](https://badge.fury.io/py/crank-vc.svg)](https://badge.fury.io/py/crank-vc)

# crank

Non-parallel voice conversion based on vector-quantized variational autoencoder with adversarial learning

## Setup

- Install Python dependency

```sh
$ git clone https://github.com/k2kobayashi/crank.git
$ cd crank/tools
$ make
```

- install dependency for mosnet

```sh
$ sudo apt install ffmpeg   # mosnet dependency
```

## Recipes
- English
    - VCC2020
    - VCC2018 (Thanks to [@unilight](https://github.com/unilight))
- Japanese
    - jsv_ver1

### Conversion samples
You can access several converted audio samples of VCC 2018 dataset in the [URL](https://k2kobayashi.github.io/crankSamples/).
- [vcc2020v1](https://drive.google.com/file/d/1uInvCwggpBYmpplYxuIOidvJkPmav8kE/view?usp=sharing)
- [vcc2018v1](https://drive.google.com/file/d/1-Z_Y9pahPQcKR0rqdhu4elI6Hz686qX6/view?usp=sharing)

## Run VCC2020 recipe

crank has prepared recipe for Voice Conversion Challenge 2020.
In crank recipe, there are 7 stages to implement non-parallel voice conversion.

- stage 0
    - download dataset
- stage 1
    - initialization
        - generate scp files and figures to be determine speaker-dependent parameters
- stage 2
    - feature extraction
        - extract mlfb and mcep features
- stage 3
    - training
- stage 4
    - reconstuction
        - generate reconstructed feature for fine-tuning of neural vocoder
- stage 5
    - evaluation
        - convert evaluation waveform
- stage 6
    - synthesis
        - synthesis waveform by pre-trained [ParallelWaveGAN](https://github.com/kan-bayashi/ParallelWaveGAN)
        - synthesis waveform by GriffinLim
- stage 7
    - objective evalution
        - mel-cepstrum distortion
        - mosnet

### Put dataset to downloads

Note that dataset is only released for the participants (2020/05/26).
```
$ cd egs/vaevc/vcc2020v1
$ mkdir downloads && cd downloads
$ mv <path_to_zip>/vcc2020_{training,evaluation}.zip downloads
$ unzip vcc2020_training.zip
$ unzip vcc2020_evaluation.zip
```

### Run feature extraction and model training

Because the challenge defines its training and evaluation set, we have initially put configuration files.
So, you need to run from 2nd stage.

```sh
$ ./run.sh --n_jobs 10 --stage 2 --stop_stage 5
```

where the ```n_jobs``` indicates the number of CPU cores used in the training.


## Configuration
Configurations are defined in ```conf/mlfb_vqvae.yml```.
Followings are explanation of representative parameters.

- feature

When you create your own recipe, be carefull to set parameters for feature extraction such as ```fs```, ```fftl```, ```hop_size```, ```framems```, ```shiftms```, and ```mcep_alpha```. These parameters depend on sampling frequency.

- feat_type

You can choose ```feat_type``` either ```mlfb``` or ```mcep```.
If you choose ```mlfb```, the converted waveforms are generated by either GllifinLim vocoder or ParallelWaveGAN vocoder.
If you choose ```mcep```, the converted waveforms are generated by world vocoder (i.e., excitation generation and MLSA filtering).

- trainer_type

We support training with ```vqvae```, ```lsgan```, ```cyclegan```, and ```stargan``` using same generator network.
  - ```vqvae```: default vqvae setting
  - ```lsgan```: vqvae with adversarial learning
  - ```cyclegan```: vqvae with adevesarial learning and cyclic constraints
  - ```stargan```: vqvae with adevesarial learning similar to cyclegan

## Create your recipe

### Copy recipe template

Please copy template directory to start creation of your recipe.

```sh
$ cp -r egs/vaevc/template egs/vaevc/<new_recipe>
$ cd egs/vaevc/<new_recipe>
```

### Put .wav files

You need to put wav files appropriate directory.
You can choose either modifying ```download.sh``` or putting wav files.
In either case, the wav files should be located in each speaker like following
```<new_recipe>/downloads/wav/{spkr1, spkr2, ..., spkr3}/*.wav```.

If you modify ```downaload.sh```,

```sh
$ vim local/download.sh
```

If you put wav files,

```sh
$ mkdir downloads
$ mv <path_to_your_wav_directory> downloads/wav
$ touch downloads/.done
```

### Run initialization

The initialization process generates kaldi-like scp files.

```sh
$ ./run.sh --stage 0 --stop_stage 1
```

Then you modify speaker-dependent parameters in ```conf/spkr.yml``` using generated figures.
Page 20~22 in [slide](https://www.slideshare.net/NU_I_TODALAB/hands-on-voice-conversion) help you how to set these parameters.


### Run feature extraction, train, reconstruction, and evaluation

After preparing configuration, you run it.

```sh
$ ./run.sh --stage 2 --stop_stage 7
```

## Citation

Please cite this paper when you use crank.

```
K. Kobayashi, W-C. Huang, Y-C. Wu, P.L. Tobing, T. Hayashi, T. Toda,
"crank: an open-source software for nonparallel voice conversion based on vector-quantized variational autoencoder",
Proc. ICASSP, 2021. (accepted)
```

## Achknowledgements

Thank you [@kan-bayashi](https://github.com/kan-bayashi) for lots of contributions and encouragement helps.

## Who we are

- Kazuhiro Kobayashi [@k2kobayashi](https://github.com/k2kobayashi) [maintainer, design and development]

- Wen-Chin Huang [@unilight](https://github.com/unilight) [maintainer, design and development]

- [Tomoki Toda](https://sites.google.com/site/tomokitoda/) [advisor]
