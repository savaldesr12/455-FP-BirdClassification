
# A Weakly-Supervised Approach to Bird Watching

[Video](https://youtu.be/SbzyTZ4xMA4)
### Weakly-Supervised Data Augmentation Network applied to the birds classification challenge.

## Problem Description:
For the final project I decided to participate in the birds classification competition on Kaggle. This dataset's main challenge was the fact that it is a Fine-Grained Image Classification. In other words the difference between classes tends to be subtle, like the shape of the beak or the feathers, unlike the difference between a car and a plane where object detection can obtain very good results. The problem for Fine-Grained Image Classification is not only applicable to species of animals, but a lot of time in real life we will encounter datasets with very subtle difference between classes, like the makes of a car and other examples where even our human eye has trouble correctly classifying objects.

## Previous Work:
Pre-trained network: InceptionV3 pretrained on the [iNaturalist 2018 dataset](https://github.com/macaodha/inat_comp_2018). I used this pre-trained network because I thought that it would be interesting to do transfer learning with a network that has been trained on a dataset with a long-tail distribution. 

WS-DAN: The paper [See Better Before Looking Closer: Weakly Supervised Data Augmentation Network for Fine-Grained Visual Classification](https://arxiv.org/abs/1901.09891). The ideas presented in this paper were specially interesting to me not only because it achieved very good results when compared to other approaches but also because of its emphasis on the data augmenation part of classification. The notion of cropping and dropping based on attention maps certainly piqued my interest. Since I was using Pytorch I found the following repository specially helpful, https://github.com/GuYuc/WS-DAN.PyTorch. I used their code for things like the data augmentation, the cropping and dropping. I followed the general structure of their WSDAN network.

## My Approach
In my approach at first I only used the InceptionV3 network and tried to use it as a feature extractor in the classification problem, i.e., only the last fully-connected layer was being affected by training, but the results weren't as good, the best validation accuracy I could get was around 60%. Then I tried using the pre-trained InceptionV3 as a weight initiliazer and the entire network was affected by training not only the last layer, the results were a lot better, it eventually achieved 81% accuracy on the test data. For my final approach I decided to use a WS-DAN, with the pre-trained InceptionV3 as the one where I get the features from (without the last fc), attention maps were generated by a weakly-supervised Basic 2D Convolution, which was supervised by a Bilinear Attention Pooling network. For the training data we feed the raw image then get its attention map and crop that part of the image and also drop it. So for 1 image we have 3 views. For the test data we only crop it using the mean of its attentions rather than a random one, here we only have 2 views of the image. For the WS-DAN model I tried using different amount of features in the inception model, 768 vs 2048 features, the 768 features performed a lot worse than the 2048, so the latter is the approach I ended up using.

![000_raw](https://user-images.githubusercontent.com/37814449/111572064-ee11cf00-8775-11eb-9ddb-d22d014d44b0.jpg)
![000_heat_atten](https://user-images.githubusercontent.com/37814449/111572068-f10cbf80-8775-11eb-92ef-3a8047481d9a.jpg)
![000_raw_atten](https://user-images.githubusercontent.com/37814449/111572056-ea7e4800-8775-11eb-9700-9cbebe59ad38.jpg)

The image on the left is the raw image that is given to the model, the middle is the heat attention map generated and the right one is approximately what will be cropped while looking closer.

## Results
After about 10 epochs the loss levels off at around 0.9, and the steps become a lot smaller. The final prediction in the test data for this model and that number of epochs is about 84%, a 3% increase from the approach only using InceptionV3.
<img width="952" alt="Screen Shot 2021-03-17 at 11 22 43 PM" src="https://user-images.githubusercontent.com/37814449/111573040-be63c680-8777-11eb-9fa3-3d9047f734fa.png">

## Discussion
### Challenges
There main challenge I encountered while implementing the WS-DAN was in the generation of the heat attention maps, at this point the loss wasn't able to decrease from 3.7, so I knew there was something wrong. Then when I tried to visualize the heat attention maps this is what I found:

![017_heat_atten](https://user-images.githubusercontent.com/37814449/111573504-a80a3a80-8778-11eb-98d2-7256af806c1b.jpg)

So it was basically using the whole image when doing the attention cropping and dropping. This happened because for the inception network I only removed the last fully-connected layer, oblivious to the fact that there was an adaptive pooling and dropout layers before this, removing them too fixed the problem. 

### Next steps
I guess if I had more time I would have tried to give the network a lot more epochs, like maybe 80. Maybe this could have allowed the attention maps that were going to be generated to approximate the shape of the birds more closely rather than encircling them more generally. I also would have tried different schedules, like exponential or cyclic, because the loss of my model would often get stuck at some loss value at around 10 epochs. 
Something else I could have tried if I had more time was generating more images like the ones in the dataset using a GAN, this could have been beneficial since it would have increased the amount of training data.

### How does your approach differ from others? Was that beneficial?
My approach differs from others in the sense that I combined a pre-trained network on the iNaturalist dataset, a dataset known for it's long-tail distribution, and the Weakly-Supervised Data Augmentation Network, a network proven to bee effective in this type of image classification. I would argue that this fusion was beneficial since it brings transfer learning on a fine-grain classification challenge dataset with a network that has shown very good results in this type of classification problems.

