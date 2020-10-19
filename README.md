## ContextNet: Improving Convolutional Neural Networks for Automatic Speech Recognition with Global Context

This repository contains TF2.x based implementation for [this paper](https://arxiv.org/pdf/2005.03191.pdf). The default setup, which is a character-based model, achieves **11.66%** and **28.31%** WERs on LibriSpeech test-clean and test-other sets respectively. These WERs can easily be improved by using:
  * large vocabulary ([subword unit](https://arxiv.org/abs/1508.07909) is one way to achieve this)
  * data augmentation ([SpecAugment](https://arxiv.org/abs/1904.08779) is one such technique)
  * regularization or limiting model capacity

### Dependencies:
  * Pysoundfile
  * Librosa
  * Tensorflow 2.x
  * [Warp-transducer](https://github.com/HawkAaron/warp-transducer)

### Task:
  * Input: Sequence of 80-dimensional filterbank features using 25msec window length and 10msec stride  
  * Output: 1K WPM
  * Dataset: 960 hours of [LibriSpeech](http://www.openslr.org/12)

### Optimization details:
  * [Adam](https://arxiv.org/abs/1412.6980) optimizer
  * Transformer LR schedule with 15K warmup steps and peak LR 0.0025
  * L2 regularization on all trainable weights
  * Variational noise added to decoder for regularization

### Architecuture details:
  * RNN-Transducer based architecuture
  * Acoustic encoder is proposed in the paper, prediction network and joint network is based on LSTM layers as used in [this paper](https://arxiv.org/abs/1811.06621)
  * Acoustic encoder:
    It consists of multiple convolution blocks (23 in all the experiments), where each block C<sub>i</sub> is made up of multiple depth-wise separable convolution layers. A convolution block is shown below with details for all 23 blocks in the following table.
    ![alt text](assets/convblock.png) 

    | Block Id   | #Conv Layers | #Output Channels | Kernel Size | Other       |
    |:----------:|:--------------:|:------------------:|:-------------:|:-------------:|
    |C<sub>0</sub> | 1            | 256 x α     | 5           | No residual |
    |C<sub>1</sub>-C<sub>2</sub>   | 5            | 256 x α     | 5           |             |
    |C<sub>3</sub> | 5            | 256 x α     | 5           | Stride is 2 |
    |C<sub>4</sub>-C<sub>6</sub>   | 5            | 256 x α     | 5           |             |
    |C<sub>7</sub> | 5            | 256 x α     | 5           | Stride is 2 |
    |C<sub>8</sub>-C<sub>10</sub>  | 5            | 256 x α     | 5           |             |
    |C<sub>11</sub>-C<sub>13</sub> | 5            | 512 x α     | 5           |             |
    |C<sub>14</sub> | 5            | 512 x α     | 5           | Stride is 2 |
    |C<sub>15</sub>-C<sub>21</sub> | 5            | 512 x α     | 5           |             |
    |C<sub>22</sub> | 1            | 640 x α     | 5           | No residual |
    
    Stride of 2 in a convolution block means last convolution layer in that block has a stride of 2, rest of them have stride of 1. SE is squeeze and excitation layer as shown below
    ![alt text](assets/SE.png) 

    3 different model variations with *global context* are shown below. The authors also experiment with context sizes of None, 256, 512 and 1024. Currently, the implementation allows either global context or no context at all.
    | Model | α | #Params(M) |
    |-------|:--------:|:------------:|
    | Small | 0.5    | 10.8       |
    | Medium| 1.0    | 31.4       |
    | Large | 2.0    | 112.7      |

  * Label encoder: Single LSTM layer with input dimension 640 and width 2048
  * (Optional/Not implemented) RNN-LM: 3 LSTM layers of width 4096

#### SpecAugment:
  * Mask parameter F = 27
  * 10 time masks with maximum time-mask ratio, p<sub>s</sub> = 0.05
  * Maximum size of the time mask p<sub>s</sub> * length-of-utterance
  * Time warping is not used

### Author's observations:
  * Swish activation, f(x) = x * σ ( β * x), works better than RELU. β = 1 is used in the paper
  * Increasing context size in SE layer improves the model performance on test-other set. Model without any context also performs very well and is comparable with model performances with non-zero context size
  * A progressive downsampling of 8 achieves good tradeoff between computational cost and model performance
  * The proposed architecture is also effective on large scale dataset
  
**Note:** All the images, tables and details are taken from the original paper unless mentioned otherwise.
