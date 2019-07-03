---
title: "AI Camp 2019 - Pose Classification"
date: 2019-07-03T12:00:24+08:00
draft: false
image: "/images/blog/ai_camp.png"
tags: ["computer vision", "ai camp", "image classification", "2019", "eugene ang"]
type: post
comments: true
---

# AI Camp - Pose Classification 

### Quick Links
- [code repository](https://github.com/DinoHub/aicamp)


## Problem
The inaugural AI Camp presented the task of pose classification to students from Junior Colleges, Polytechnics, and Universities. An in-house image dataset of 15 poses was collected by the competition architects for the competition. The poses ranged from full-body poses such as the yoga Warrior Pose and Kung Fu Crane, to hand poses such as Spiderman and Handshake. The goal of the competition was to build a deep learning method which could learn how to classify the poses in a generalised way to achieve high classification accuracy on unseen images of the poses.

![Sample Image of Poses](/images/blog/sample.png 'Sample Pose Images of Korean Heart and Spiderman')

A week prior to the start of the camp, participants were given basic training on deep learning methods for image classification, and a training and validation set of 11 poses were released. The participants had one week to come up with their own methods to perform the pose classification. On the first day of the camp, the participants were given a test set they had never encountered before to check the performance of their methods. Thereafter, a curveball was released to them with an additional four poses to classify. The participants were given another test set to evaluate the performance of their methods on all 15 classes. Finally, the camp was rounded off with an evaluation against a final test set. The finalists were chosen based on the teams' performances on the final test set. The finishing order for the finalists was eventually decided after the finalists presented their methods in a pitch to the judges.

As the competition architects, Senior Engineer Eugene Ang and Engineer Ling Evan from Digital Hub prepared their own solution to the task.

## Solution 
A combination of multiple methods were applied to deep learning to obtain optimal results.

### Data Augmentation 
In order to increase the size and variety of images in the dataset, data augmentation was applied. During the training of the deep learning model, random augmentations were applied to the training images. The brightness of the images were randomly increased and decreased, and the images were zoomed in and out, sheared and cropped randomly. This meant that the model would train on slightly different images each epoch and would learn to generalise and only pick up on key features that defined a class. Data augmentation essentially increases the size of the training set and provides the deep learning model with more "experience" with images that may more closely resemble images from the test set. This can provide several percentage points of accuracy increase.

#### Test-time augmentation
Test-time augmentation is a variant on data augmentation and ensembling. Instead of augmenting the images at training time, images are augmented at test time while being fed into the model. The same minor augmentations are applied of brightness, zooming, shearing, and cropping in set controlled amounts to each image. This process is repeated multiple times with different augmentations applied. The predictions for the image are aggregated and the class with the highest confidence is chosen as the predicted class. This process reaps the benefits of ensembling and instead of varying the model used to predict the test images, the test images are varied. This method can provide a slight increase in training accuracy.

### Pretrained model
Deep Convolutional Neural Networks are used to perform feature extraction and image classification. These neural networks can be quite large with upwards of one hundred layers. Networks with good architecture for sufficient performance are deep and are extremely tedious to build and train from scratch. As such, pretrained models are used. Models such as ResNet50 and Xception were used as they are relatively compact and provide excellent image classification performance. A pretrained model trained on imagenet is taken with its weights and used as a base model for the custom image classification task. The final few layers of the pretrained models are removed and a Global Average Pooling layer and a final Dense layer with the corresponding number of nodes as classes in the custom dataset are added. For this task, an output dense layer of size 11 would be built first, then discarded and a new layer of size 15 added after the curveball challenge. In this configuration, the model is fine-tuned for around 20 epochs to achieve the best accuracy on the new custom dataset. Early layers may be frozen to prevent their weights from being updated during the backpropagation phase of training. This allows the training time to be drastically shortened due to the reduced number of parameters, while at no cost to performance as theses early layers extract general features and are unlikely need much changing. The use of pretrained models can reduce the amount of time and resources taken to classify a new custom dataset while providing good performance as the model is already trained to extract features.

### Human detection
In order to make sure the deep learning models are learning features of the human doing the pose and not the background, human detection is deployed to remove the background by creating a tight crop around the human. You-only-look-once (YOLOv3) was used for this task. YOLOv3 is an object detector and classifier that identifies and object and draws a bounding box around it. It is also trained to classify the identified object. YOLOv3 was modified to show only the largest person object identified in each image, which is assumed to be the person performing the pose. The bounding boxes are cropped out to create a training dataset with a tight crop around the human performing the pose. This method prevented the deep learning models from learning irrelevant features in the background and can provide a slight increase in training accuracy.

### Ensembling
Ensembling is the aggregation of multiple deep learning models. Several models are trained with slight differences. Different model architectures such as ResNet50 and Xception, with different cross-validation splits, and different training hyperparameters such as optimizer and learning rate are trained on the training set. During testing, each model performs the test and produces a softmax prediction vector for each test image. In a simple average, the predictions for each image from each model are summed and the class with the highest total confidence is selected as the predicted class. This method leverages on the idea that different models classify different images correctly and the collective wisdom of the ensemble allows it to perform prediction with a higher accuracy than any individual model. To further improve the ensemble, a selective ensemble of the best-performing models can be taken. A grid-search method was created by a Digital Hub intern Ivan to iterate through all combinations of available models to find the best possible combination to ensemble.

### Model Distillation 
The ensemble method provides an accuracy boost to the task, but comes at a cost of being very bulky and slow to run. Several models have to perform the prediction on the test set, and the ensemble would consume a significant amount of memory and time to run. As such, it is beneficial to have a lighter model with the performance of the ensemble. Model distillation is used to creating training data for such a model. After an ensemble is created, each training image is tested with the ensemble to obtain the aggregate prediction softmax vector for that image. The soft vector is then used as the ground truth instead of a hard one-hot encoded vector for the distilled model. The model trained with the soft ground truth labels can reach closer to the performance of the ensemble than the those trained on the conventional hard one-hot encoded labels. This is because there are inherent similarities in certain poses, such as HandGun and Spiderman, that should reflect in the prediction confidence that a hard label does not encapsulate. This method can produce ensemble-like performance in a smaller, lighter, single-model package.

