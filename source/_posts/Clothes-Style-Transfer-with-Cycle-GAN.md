---
title: Clothes Style Transfer with Cycle GAN
catalog: true
date: 2017-11-27 21:52:52
subtitle:
header-img:
tags:
---

## Our Goal ##

**What is our goal**

- Detect the clothes in a picture and change its texture and color
- Changing the clothes will not cause drastic change towards the skin and hairs of a person.

**What is not our goal**

- Changing the clothes does reflect the size of the clothes(eg, S, M, L, XL).


## Our data ##

- Clothes data: shared by Fashion Image Synthesis
- Segmentation data: ???


## Our methods and results ##

### Method 1: The original cycle GAN implementation ###

This yeilds not so satisfactory result due to the lack of difference between the two training groups. Unlike the picture <-> monnet or
horse <-> zebra training, in which great difference and pattern could be found among the two training sets, in our case it is not so easy for the
model to identify the difference between pants and shorts.

Another reason for unsatisfactory result is that the area that we want to
perform style transfer on is usually too detailed for cycle GAN, which is good at pixel to pixel transfer for two sets of images.


### Method 2: The skin_loss loss function ###

We used a simple algorithm to detect skin on the picture and then run cycle GAN with this new loss function. The basic idea is that when we transfer from pants to shorts the skin area decreases and grows when going the other way. The new loss function is supposed to be able to give some sort of 'hint' to the discriminator on which way to go.

We had better result with this loss function. However, there are also several disadventages of using this loss function.

The first is that this is a very rough skin detection algorithm and does not give very accurate 'hint' to the discriminator. Therefore in some cases the loss function, rather than giving hints, actually confuses the discriminator and generator.

Another one is that this loss function is too specific to our cases. We are training between pants and shorts, which has the great attribute that skin area changes drastically. If there is no such property for other training pairs, or if this other property is very hard to represent, our method will not work anymore.

### Method 3: Giving hint directly to the discriminator ###

Inspired by the encoding-decoding image segmentation algorithm, we introduced the third method that gives hint directly to the discriminator. In the encoding-decoding image segmentation algorithm, whenever the network is unsampling a layer, the original mapping before pooling is also added to the layer. Since during the encoding stage, certain amount of semantic meanings of the original image is preserved, the unsampled layer will preserve more infromation about the image, which helps to make decoding performs better.

Therefore, in our case we used an image segmentation alroithm to segment our the area that we want the network to focus on, and then changed the input of the discriminator to

I' = I + \lambda*S(I)*I

Where I is the original input, S(I) is the segmented map and lambda is a hypervariable.

**This method need further experiment**

### Method 4: Loss function in more sophisticated way ###
  
This method addresses the first disadvantage mentioned in Method 2. Instead of using a very rough loss function , we used segmentation mapping to differentiate between clothes and skin.

**This method need further experiment**