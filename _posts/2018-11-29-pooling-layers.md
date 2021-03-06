---
layout:     post
title:      池化层
keywords:   博客
categories: [机器学习]
tags:	    [最大池化，平均池化]
---

在卷积神经网络中，通常会使用池化层来减少展示量，以此提高计算速度，并强化一些特征的检测功能。本文将通过几个例子来了解池化层背后的机制，以及为什么要使用池化层。

## 最大池化

假设我们有一个4x4的输入，使用最大池化(max pooling)后的输出，是一个2x2的输出。这个例子中将输入划分为4个不同的区域，赋予不同的颜色，每个输出值是其对应颜色区域的最大值。max pooling的超参数是过滤器的大小2，步长2。

  ![](/images/images_2018/11-29_01.png)  

### 最大池化背后的机制

我们把4x4的区域看作是某个特征的集合，即神经网络某个层中的激活状态，一个大的数字意味着或许检测到了一个特定的特征。所以max pooling做的是，在任何地方检测到了特征，就保留最大的数值。大家使用max pooling的主要原因是在很多实验中发现它的效果很好。这是一类最常用的池化。

max pooling的一个有趣的特性是，它有一套超参，但是它没有任何参数需要学习。即一旦确定了f和s，就确定了计算。

### 计算最大池化输出的公式

用来计算卷积层输出大小的公式，同样适用于max pooling。公式是n加上2p减去f除以s再加1。第二个例子是一个5x5的输入，应用了一个3x3的过滤器，于是f=3,s=1，输出的大小是3x3。

  ![](/images/images_2018/11-29_02.png)   

需要注意的是，在多维情况下，即有N个通道时，max pooling在每个通道上是独立进行的，输入有多少通道，输出就有多少通道。

## 平均池化

平均池化(Average pooling)不是在取最大值，而是取平均值。最大池化比平均池化使用得多得多。唯一的例外是有时候在深度非常大的神经网络，也许可以使用平均池化来合并表示。

  ![](/images/images_2018/11-29_03.png)    

## 总结  

池化最常用的超参数是，f=2,s=2或者f=3,s=2。当max pooling时，通常情况下是不进行任何的补位，即p=0。当f=2,s=2时，效果相当于将表示的高度和宽度缩小了2倍。

  ![](/images/images_2018/11-29_04.png)   


  
