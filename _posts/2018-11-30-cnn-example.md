---
layout:     post
title:      卷积神经网络实例
keywords:   博客
categories: [机器学习]
tags:	    [卷积神经网络，池化]
---

在卷积神经网络中，通常会使用池化层来减少展示量，以此提高计算速度，并强化一些特征的检测功能。本文将通过几个例子来了解池化层背后的机制，以及为什么要使用池化层。

## 一个简单的CNN实例

下图是一个非常典型的ConvNet的例子。卷积神经网络设计中的许多工作是选择各种超参数，比如步长多少,padding多少，过滤器多少等。需要记住的一件事是：

随着神经网络越来越深，图像的高度和宽度会逐渐变小，在这个例子中是从39到37到17到7，这是一种常见的趋势。实际上，一个典型的ConvNet通常有三种类型的层，卷积层(Conv)，池层和完全连接层(FC)。虽然我们可能只使用卷积层设计一个相当好的神经网络，但大多数神经网络架构也会有池层和完全连接层。

  ![](/images/images_2018/11-30_01.png)  

## 一个识别手写数字的CNN实例

假设输入图像是32x32x3的一个RGB图像，受到经典的神经网络LeNet-5的启发，用5x5的过滤器，步长为1，使用8个过滤器，则第一层输出是28x28x8，我们称这一层为Conv1，接下来使用一个最大池化层Pool1，超参数f=2，s=2，原有的高度和宽度将会缩小一半，输出为14x14x8。（注意下图有错误，画的是6个过滤器，实际应该是8个）

  ![](/images/images_2018/11-30_02.png)  

神经网络中当人们说到网络层数的时候，通常指那些有权重，有参数的网络层数量，因为池化层没有权重，没有参数，我们会使用Conv1和Pool1为一体的说法，把它们作为层一。在14x14x8的基础上，再做一层卷积，使用5x5的过滤器，步长为1，共16个过滤器，得到一个10x10x16的单元，称为Conv2，继续做最大池化，使用参数f=2，s=2，得到5x5x16的输出，称为Pool2，Conv2和Pool2合称为神经网络层二。

  ![](/images/images_2018/11-30_03.png) 

我们把池化层二展开成一个400x1的向量，用这400个单元做输入，创建一个有120个单元的下一层，这是我们的第一个全连接网络层，称之为FC3，这一层400个输入单元和120个输出单元密集相连。FC3的权重矩阵大小为120x400，并有一个偏差参数，是一个120的向量。最后在120的单元基础上再加一个全连接层FC4，获得用于softmax层的84个单元，如果是识别手写数字，那softmax层就有10个输出单元。这就是一个相对典型的例子，展示了一个卷积神经网络的大致构成。

  ![](/images/images_2018/11-30_04.png) 

### 卷积神经网络的超参数

一个常用的法则是，不要试着创造你自己的超参数组，而是查看文献，看看其他人使用的超参数，从中选一组适用于其他人的超参数，很可能它也适用于你的应用。

### 另一种常见的神经网络模型

 
 - 一个或多个卷积层接着一个池化层；
 - 再接着一个或多个卷积层叠加一个池化层；
 - 然后叠加几个全连接层；
 - 也许最后还叠加一个softmax层。

## 回顾

我们通过下图回顾一下神经网络的一些细节，比如激活输入的尺寸，比如激活输入a0的大小为3072，输入层是没有参数的。多数参数在神经网络的全连接层上。同时激活输入大小也逐渐变小。如果减少的太快，通常不利于网络性能。以上例子中首先大小从6272减到1600，接下来慢慢减小到84，知道最终的softmax层输出。

  ![](/images/images_2018/11-30_05.png) 

如何利用神经网络的基本构件：卷积层、池化层和全连接层，来构建一个有效的神经网络，最好的方法就是学习一定数量的实例，看看人家是如何做的，从中获得灵感。