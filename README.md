# Working Version of iSeeBetter: Spatio-Temporal Video Super Resolution using Recurrent-Generative Back-Projection Networks Pytorch implementation

---

### PDF: **[arXiv](https://arxiv.org/abs/2006.11161)** 


## Required Packages

```
torch==1.4
pytorch-ssim==0.1
numpy==1.16.4
scikit-image==0.15.0
tqdm==4.37.0
```

Step 0: To load the required Python modules:
```bash
pip3 install -r requirements.txt
```
or
```bash
conda install --file requirements.txt
```

Compile [Pyflow](https://github.com/pathak22/pyflow) which is a Python wrapper for [Ce Liu's C++ implementation](https://people.csail.mit.edu/celiu/OpticalFlow/) of Coarse2Fine Optical Flow.
Pyflow binaries that we used have been built for Ubuntu and macOS with Python 3.7 and are available in the repository.
If you need to rebuild Pyflow, (i) simply follow the instructions below or (ii) refer to the [Pyflow Git](https://github.com/pathak22/pyflow)

Step 1: Build Pyflow:
```bash
cd pyflow/
python setup.py build_ext -i  # build pyflow
python demo.py                # to make sure pyflow works
cp pyflow*.so ..
```

Step 2: Train or test iSeeBetter using the instructions in the relevant sections below.


## Overview

Recently, learning-based models have enhanced the performance of single-image super-resolution (SISR). However, applying SISR successively to each video frame leads to a lack of temporal coherency. Convolutional neural networks (CNNs) outperform traditional approaches in terms of image quality metrics such as peak signal to noise ratio (PSNR) and structural similarity (SSIM). However, generative adversarial networks (GANs) offer a competitive advantage by being able to mitigate the issue of a lack of finer texture details, usually seen with CNNs when super-resolving at large upscaling factors. We present iSeeBetter, a novel GAN-based spatio-temporal approach to video super-resolution (VSR) that renders temporally consistent super-resolution videos. iSeeBetter extracts spatial and temporal information from the current and neighboring frames using the concept of recurrent back-projection networks as its generator. Furthermore, to improve the "naturality" of the super-resolved image while eliminating artifacts seen with traditional algorithms, we utilize the discriminator from super-resolution generative adversarial network (SRGAN). Although mean squared error (MSE) as a primary loss-minimization objective improves PSNR/SSIM, these metrics may not capture fine details in the image resulting in misrepresentation of perceptual quality. To address this, we use a four-fold (MSE, perceptual, adversarial, and total-variation (TV)) loss function. Our results demonstrate that iSeeBetter offers superior VSR fidelity and surpasses state-of-the-art performance.
 
![adjacent frame similarity](https://github.com/amanchadha/iSeeBetter/blob/master/images/iSeeBetter_AFS.png)
<p align="center">Figure 1: Adjacent frame similarity</p>
 
![network arch](https://github.com/amanchadha/iSeeBetter/blob/master/images/iSeeBetter_NNArch.jpg)
<p align="center">Figure 2: Network architecture</p>

## Model Architecture

Figure 2 shows the iSeeBetter architecture that consists of and SRGAN  as its generator and discriminator respectively. RBPN has two approaches that extract missing details from different sources, namely SISR and MISR. Figure 3 shows the horizontal flow (represented by blue arrows in Fig. 2) that enlarges LR(t) using SISR. Figure 4 shows the vertical flow (represented by red arrows in Figure 2) which is based on MISR that computes residual features from a pair of LR(t) and its neighboring frames (LR(t−1), ..., LR(t−n) coupled with the pre-computed dense motion flow maps (F(t−1), ..., F(t−n)). At each projection step, RBPN observes the missing details from LR(t) and extracts residual features from neighboring frames to recover details. Within the projection models, RBPN utilizes a recurrent encoder-decoder mechanism for fusing details extracted from adjacent frames in SISR and MISR and incorporates them into the estimated frame SR(t) through back-projection. Once an SR frame is synthesized, it is sent over to the discriminator (shown in Figure 5) to validate its "authenticity". 

![ResNet_MISR](https://github.com/amanchadha/iSeeBetter/blob/master/images/ResNet_MISR.jpg)
<p align="center">Figure 3: ResNet architecture for MISR that is composed of three tiles of five blocks where each block consists of two convolutional layers with 3 x 3 kernels, stride of 1 and padding of 1. The network uses Parametric ReLUs for its activations.</p>

![DBPN_SISR](https://github.com/amanchadha/iSeeBetter/blob/master/images/DBPN_SISR.png)
<p align="center">Figure 4: DBPN architecture for SISR, where we perform up-down-up sampling using 8 x 8 kernels with stride of 4, padding of 2. Similar to the ResNet architecture above, the DBPN network also uses Parametric ReLUs as its activation functions.</p>

![Disc](https://github.com/amanchadha/iSeeBetter/blob/master/images/Disc.jpg)
<p align="center">Figure 5: Discriminator Architecture from SRGAN. The discriminator uses Leaky ReLUs for computing its activations.</p>

![iSB_Loss](https://github.com/amanchadha/iSeeBetter/blob/master/images/iSeeBetter_Loss.png)
<p align="center">Figure 6: The MSE, perceptual, adversarial, and TV loss components of the iSeeBetter loss function</p>

## Dataset

To train iSeeBetter, we amalgamated diverse datasets with differing video lengths, resolutions, motion sequences and number of clips. Table 1 presents a summary of the datasets used. When training our model, we generated the corresponding LR frame for each HR input frame by performing 4x down-sampling using bicubic interpolation. We also applied data augmentation techniques such as rotation, flipping and random cropping. To extend our dataset further, we wrote scripts to collect additional data from YouTube, bringing our dataset total to about 170,000 clips which were shuffled for training and testing. Our training/validation/test split was 80\%/10\%/10%.

Get the [SPMCS and Vid4 dataset](https://drive.google.com/drive/folders/1sI41DH5TUNBKkxRJ-_w5rUf90rN97UFn?usp=sharing) and the [Vimeo90K dataset](http://data.csail.mit.edu/tofu/dataset/vimeo_septuplet.zip). You can also use ```DatasetFetcher.py``` to get Vimeo90K.

![results](https://github.com/amanchadha/iSeeBetter/blob/master/images/Dataset.jpg)
<p align="center">Table 1. Datasets used for training and evaluation</p>

## Results

We compared iSeeBetter with six state-of-the-art VSR algorithms: DBPN, B123 + T, DRDVSR, FRVSR, VSR-DUF and RBPN/6-PF.

![results2](https://github.com/amanchadha/iSeeBetter/blob/master/images/Res1.jpg)
<p align="center">Table 2. Visually inspecting examples from Vid4, SPMCS and Vimeo-90k comparing RBPN and iSeeBetter. We chose VSR-DUF for comparison because it was the state-of-the-art at the time of publication. Top row: fine-grained textual features that help with readability; middle row: intricate high-frequency image details; bottom row: camera panning motion.</p>

![results1](https://github.com/amanchadha/iSeeBetter/blob/master/images/Res2.jpg)
<p align="center">Table 3. PSNR/SSIM evaluation of state-of-the-art VSR algorithms using Vid4 for 4x. Bold numbers indicate best performance.</p>

## Pretrained Model
Model trained for 4 epochs included under ```weights/```

## Usage

### Training 

Train the model using:

```python3 iSeeBetterTrain.py```

(takes roughly 1.5 hours per epoch with a batch size of 2 on an NVIDIA Tesla V100 with 16GB VRAM)

### Testing

To use the pre-trained model and test on a random video from within the dataset:

```python3 iSeeBetterTest.py --upscale_only```

## Acknowledgements

Credits:
- [SRGAN Implementation](https://github.com/leftthomas/SRGAN) by LeftThomas.
- We used [RBPN-PyTorch](https://github.com/alterzero/RBPN-PyTorch) as a baseline for our Generator implementation.
