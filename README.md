# BicycleGAN-pytorch
__Pytorch__ implementation of [BicycleGAN : Toward Multimodal Image-to-Image Translation](https://arxiv.org/abs/1711.11586). This model can generate diverse images from an input image using random latent vector z.

## Result
### Edges2Shoes

## Model description
<p align="center"><img width="100%" src="png/bicyclegan.png" /></p>  

### cVAE-GAN
This can be seen as __image reconstruction process.__ By doing this, we can make the encoder extract proper latent code z which specializes given image 'B' and the generator generate an image which has features of 'B'. Of course, the generator also needs to be able to fool the discriminator. Futhermore, It uses KL-divergence to make the generator be able to generate images using randomly sampled z from normal distribution.

### cLR-GAN
This can be seen as __latent code reconstruction process.__ The main purpose of this process is to make invertible mapping between B and z. It leads to bijective consistency between latent encoding and output modes that is important to prevent from __mode collapse.__ If many latent codes correspond to an output mode, this is mode collapse.

## Prerequisites
* [Python 3.5+](https://www.continuum.io/downloads)
* [PyTorch 0.2.0](http://pytorch.org/)

## Training step  
1. Optimize D  
* Optimize D in cVAE-GAN using real_B and fake_B made with encoded_z(Adversarial loss).  
* Optimize D in cLR-GAN using real_B and fake_B made with random_z(Adversarial loss).  

2. Optimize G or E
* Optimize G and E in cVAE-GAN using fake_B made with encoded_z(Adversarial loss).
* Optimize G and E in cVAE-GAN using real_B and fake_B made with encoded_z(Image reconstruction loss).  
* Optimize E in cVAE-GAN using the encoder outputs, mu and log_variance(KL-div loss).  
* Optimize G in cLR-GAN using fake_B made with random_z(Adversarial loss).  

3. Optimize ONLY G(Do not update E)
* Optimize G in cLR-GAN using random_z and the encoder output mu.

## Implementation details

* __Multi discriminator__  
First, two discriminators are used for two different last output size(PatchGAN), 14x14 and 30x30. Second, two discriminators again for images made with encoded z(cVAE-GAN) and random z(cLR-GAN). Totally, __four discriminators__ are used, __(cVAE-GAN, 14x14), (cVAE-GAN, 30x30), (cLR-GAN, 14x14) and (cLR-GAN, 30x30).__

* __Conditional discriminator__  
pass

* __Encoder__  
__E_ResNet__ is used, __not E_CNN__. Residual block in the encoder is slightly different with the usual one. Check ResBlock class and Encoder class in model.py.

* __How to inject the latent code z to the generator__  
Just inject __only to the input__, not to all intermediate layers

* __Training data__  
Batch size is 1 for each cVAE-GAN and cLR-GAN which means that get two images from the dataloader and distribute to cVAE-GAN and cLR-GAN.

* __How to encode with encoder__  
Encoder returns mean and log(variance). Reparameterization trick is used, so __encoded_z = random_z * std + mean__ such that __std = exp(log_variance / 2).__

* __How to calculate KL divergence__  
Images should be here

* __How to reconstruct z in cLR-GAN__  
We get mu and log(variance) as outputs from the encoder in cLR-GAN. Use __L1 loss between mu and random_z__, not encoded_z and random_z because the latter loss can be unstable if std is big. You can check [here](https://github.com/junyanz/BicycleGAN/issues/14).
