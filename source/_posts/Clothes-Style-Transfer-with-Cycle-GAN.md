---
title: Clothes Style Transfer with Cycle GAN - Adding the role of teacher into the discriminator-generator network
catalog: true
date: 2017-11-27 21:52:52
subtitle:
header-img:
tags:
---
## Introduction ##

Cycle GAN is a very powerful tool on transfering images between two drastically different sets of images. However, if we generalize the task to similar datasets and more specific task, i.e. people wearing long pants and people wearing short pants, Cycle GAN did not perform as well.

In our project we tried to introduce the role of an external 'teacher', who gives aditional hint to the generator or the discriminator to do the task better.

## Our Goal ##

**What is our goal**

- Detect the clothes in a picture and change its texture and color
- Changing the clothes will not cause drastic change towards the skin and hairs of a person.

**What is not our goal**

- Changing the clothes does reflect the size of the clothes(eg, S, M, L, XL).


## Our data ##

- Clothes data: In Shop Clothes Retrieval Benchmark by Deep Fashion from CUHK
- Segmentation data: HumanParsing Dataset from [https://github.com/lemondan/HumanParsing-Dataset](https://github.com/lemondan/HumanParsing-Dataset)

## Our code
- We used the cycle GAN implementation completed by Van Huy (Available [here](https://github.com/vanhuyz/CycleGAN-TensorFlow))
- The our code for training is available [here](https://github.com/CSCI599-Lim-Squad/clothesGAN)


## Our methods and results ##

### Method 1: The original cycle GAN implementation (No teacher)

![CycleGAN](cycle_gan_graph.jpg)

This yeilds not so satisfactory result due to the lack of difference between the two training groups. Unlike the picture <-> monnet or horse <-> zebra training, in which great difference and pattern could be found among the two training sets, in our case it is not so easy for the model to identify the difference between pants and shorts.

Another reason for unsatisfactory result is that the area that we want to perform style transfer on is usually too detailed for cycle GAN, which is good at pixel to pixel transfer for two sets of images.

**CycleGAN Result**

![CycleGAN Result](cycle_gan_result.png)

### Method 2: The skin_loss loss function (Generator's Teacher)

![Skin Loss CycleGAN](skin_loss_graph.jpg)

**Skin Loss Functions**

![Skin Loss Formula](skin_loss_formula.png)

We used a simple algorithm to detect skin on the picture and then run cycle GAN with this new loss function. The basic idea is that when we transfer from pants to shorts the skin area decreases and grows when going the other way. The new loss function is supposed to be able to give some sort of 'hint' to the discriminator on which way to go. Specifically in our model X represents long pants and Y represents short pants.

The algorithm detects skin area by its RGB value. The condition we used in our model is R>95 and G>40 and B>20. The S function in the above equation represents such classification function.

We then used experiments to find the value proper for constructing the logits of Y, and we found out that the result achieved by 229 yeilds the best result.

We had better result with this loss function. However, there are also several disadventages of using this loss function.

The first is that this is a very rough skin detection algorithm and does not give very accurate 'hint' to the discriminator. Therefore in some cases the loss function, rather than giving hints, actually confuses the discriminator and generator.

Another one is that this loss function is too specific to our cases. We are training between pants and shorts, which has the great attribute that skin area changes drastically. If there is no such property for other training pairs, or if this other property is very hard to represent, our method will not work anymore.

**Skin Loss Result**

![Skin Loss Result](skin_loss_result.png)

### Method 3: Giving hint directly to the discriminator (Discriminator's Teacher)

![Segmentation CycleGAN](segmentation_graph.jpg)

**Segmentation Generative Loss Function**

![Segmentation Generative Loss](segmentation_generative_loss.png)

Inspired by the encoding-decoding image segmentation algorithm, we introduced the third method that gives hint directly to the discriminator. In the encoding-decoding image segmentation algorithm, whenever the network is unsampling a layer, the original mapping before pooling is also added to the layer. Since during the encoding stage, certain amount of semantic meanings of the original image is preserved, the unsampled layer will preserve more infromation about the image, which helps to make decoding performs better.

Therefore, in our case we used an image segmentation alroithm to segment our the area that we want the network to focus on, and then changed the input of the discriminator to help discriminator.

The Seg function in the equation represents the Segmentation network. It will return a mapping of pants and legs labelled with 1 and otherwise 0 of the original image. Our implementation of segmentation network takes a typlical encoding-decoding architecture and followed the that from the project completed by Jakub Naplava (could be found on https://github.com/arahusky/Tensorflow-Segmentation)

This did not turn out to be a great method to do the task. The generated results were even worth than the results without such boost to the discriminator.

![Segmentation Formula](segmentation_formula.png)

Where I is the original input, S(I) is the segmented map and lambda is a hypervariable.

This did not turn out to be a great method to do the task. The generated results were even worth than the results without such boost to the discriminator.

**Segmentation Result**

![Segmentation Result](segmentation_result.jpg)

### Method 4: Loss function in more sophisticated way (Additional Tutoring for Generator)

![Segmentation & Skin Loss CycleGAN](skin_loss_with_segmentation_graph.jpg)

**Skin Loss with Segmentation**

![Skin Loss with Segmentation](skin_loss_with_segmentation.png)
  
This method addresses the first disadvantage mentioned in Method 2. Instead of using a very rough loss function , we used segmentation mapping to differentiate between clothes and skin.

The Seg function in the equation is similar to that in the previous part. The only difference is that this seg function returns a mapping only to leg exposed to the air.

The results did not trun out to be better than the results achieved by method 2 in most cases.

**Skin Loss with Segmentation Result**

![Skin Loss with Segmentation](skin_loss_with_segmentation_result.png)

### Discussions

Our results proved that it is possible to add a loss function as a teacher to cycle GAN in order to have better results. Although we have only been successful on adding a loss function to the generator, it is not impossible to introduce a teacher to the discriminator of for Cycle GAN.

We also encountered the interesting phenomenon that more specific loss function does not always give better results. In our opinion, this phenomenon might have been caused by the weakness of the more specific loss function. A more specific loss function means that it will focus on a smaller part of image and thus gives weaker gradients than other loss functions.

## Acknowledgements ##

**Human Parsing**

Deep Human Parsing with Active Template Regression, Pattern Analysis and Machine Intelligence, IEEE Transactions on, Xiaodan Liang and Si Liu and Xiaohui Shen and Jianchao Yang and Luoqi Liu and Jian Dong and Liang Lin and Shuicheng Yan

Pattern Analysis and Machine Intelligence, IEEE Transactions on, ICCV, Xiaodan Liang and Chunyan Xu and Xiaohui Shen and Jianchao Yang and Si Liu and Jinhui Tang and Liang Lin and Shuicheng Yan

**In Shop Retrieval**

DeepFashion: Powering Robust Clothes Recognition and Retrieval with Rich Annotations, Proceedings of IEEE Conference on Computer Vision and Pattern Recognition (CVPR), Ziwei Liu, Ping Luo, Shi Qiu, Xiaogang Wang, and Xiaoou Tang
