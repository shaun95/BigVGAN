# BigVGAN: A Universal Neural Vocoder with Large-Scale Training
![image](https://user-images.githubusercontent.com/56749640/173193781-0ee419a5-df66-4b94-8585-546167ecceb5.png)


In this repository, I try to implement BigVGAN (specifically BigVGAN-base model) [[Paper]](https://arxiv.org/pdf/2206.04658.pdf) [[Demo]](https://bigvgan-demo.github.io/).

## Pre-requisites
0. Pytorch >=3.9 and torchaudio >= 0.9

0. Download datasets
    1. Download the VCTK dataset
    

## Training Exmaple
```sh
# VCTK
python preprocess.py

python train_bigvgan_vocoder.py -c configs/vctk_bigvgan.json -m bigvgan
```

## 2022-06-12
- Current ver has some redundant parts in some modules (e.g., data_utils have some TTS module. Ignore it plz)

## BigVGAN vs HiFi-GAN 

1. Leaky Relu --> x + (1/a)*sin^2(ax)

2. MRF --> AMP block 
- Up --> Low-pass filter --> Snake1D --> Down --> Low-pass filter
- I need to review this module. I used torchaudio to implement up/downsampling with low-pass filter. I used the rolloff value of 0.25 in the T.resample but I'm not sure this value is equal to sr/(2*m) where m is 2.
- torchaudio > 0.9 is needed to use the rolloff parameter for anti-aliasing in T.resample
- There are some issues of STFT function in pytorch of 3.9 (When using mixed precision, waveform need to be changed to float before stft ) 

3. MSD --> MRD (Univnet discriminator) [UnivNet unofficial github](https://github.com/mindslab-ai/univnet/blob/9bb2b54838bb6d7ce767131cc7b8b61198bc7558/model/mrd.py#L49)

4. max_fre 8000 --> 12,000 for universal vocoder (sr: 24,000)
- We use the sampling rate of 22,050 and linear spectrogram for input speech.

## Low-pass filter
- [ ] pytorch extension ver with scipy??? [StarGAN3](https://github.com/NVlabs/stylegan3/blob/b1a62b91b18824cf58b533f75f660b073799595d/training/networks_stylegan3.py)
- [X] torchaudio ver (0.9 >=) can use Kaiser window and rolloff (cutoff frequency)

[Torchaudio tutorial](https://tutorials.pytorch.kr/beginner/audio_resampling_tutorial.html)

torchaudio.transforms is much faster than torchaudio.functional when resampling multiple waveforms using the same paramters. 


```sh
# beta 
m = 2
n = 6
f_h = 0.6/m
A = 2.285*((m*n)/2 - 1)*math.pi*4*f_h+7.95
beta = 0.1102*(A-8.7)
4.663800127934911
```

## Results (~ing)
![image](https://user-images.githubusercontent.com/56749640/173265977-f77d6e54-f723-4547-a29c-b669b43f47cb.png)
[Audio](https://github.com/sh-lee-prml/BigVGAN/tree/main/audio)

 I train the BigVGAN-base model with batch size of 64 (using two A100 GPU) and an initial learning rate of 2 × 10<sup>−4</sup>

(In original paper, BigVGAN (large model) uses batch size of 32 and an initial learning rate of 1 × 10<sup>−4</sup>
 to avoid an early training collapse)

For the BigVGAN-base model, I have not yet experienced an early training collapse with batch size of 64 and an initial learning rate of 2 × 10<sup>−4</sup>


## Automatic Mixed Precision (AMP)

The original paper may not use the AMP during training but this implementation includes AMP. Hence, the results may be different in original setting.

For training with AMP, I change the dtype of representation to float for torchaudio resampling (Need to be changed for precise transformation).

## Reference
- VITS: https://github.com/jaywalnut310/vits
- UnivNET: https://github.com/mindslab-ai/univnet
