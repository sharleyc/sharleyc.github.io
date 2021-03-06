---
layout:     post
title:      CAM论文初探
keywords:   博客
categories: [机器学习]
tags:	    [CAM, 类激活图, 可视化]
---

类激活图(CAM,class activation map)可视化，是指对输入图像生成类激活的热力图。类激活图是与特定输出类别相关的二维分数网格，对任何输入图像的每个位置都要进行计算，它表示每个位置对该类别的重要程度。以猫狗分类举例来说，CAM可以生成类别“猫”/“狗”的热力图，表示图像的各个部分与“猫”/“狗”的相似程度。

   ![](/images/images_2018/11-10_01.jpg)  

## CAM的具体实现
 
CAM有助于了解一张图片的哪一个部分让卷积神经网络作出了最终的分类决策，因此CAM在出现分类错误的情况下，能帮助进行调试。具体的实现方式是"Grad-CAM: visiual explanations from deep networks via gradient-based localization"这篇论文中描述的方法。这种方法非常简单：给定一张输入图像，对于一个卷积层的输出特征图，用类别相对于通道的梯度对这个特征图中的每个通道进行加权。直观上来看，理解这个技巧的一种方法是，你使用“每个通道对类别的重要程度”对“输入图像对不同通道的激活强度”的空间图进行加权，从而得到了“输入图像对类别的激活强度”的空间图。

### 使用VGG16模型预测测试图片

如下所示，测试图片是一张非洲象： 

   ![](/images/images_2018/11-12_01.jpg)  

运行以下代码得到测试图片类别的索引编号为386： 

    # 测试图片
    img_path = "F:\\elephant.jpg"
    # 加载带有预训练权重的VGG16网络
    model = VGG16(weights='imagenet')
    # 将测试图片处理成(1,224,224,3)形状的张量
    x = img_to_senor(img_path, (224,224))
    # 预测图片类型
    preds = model.predict(x) 
    # 获取预测值最大的类别的索引编号
    max_index=np.argmax(preds[0])
    print(max_index)

### 应用Grad-CAM算法


  
## 迁移学习的实现方法  

迁移学习中一种常用且非常高效的方法是使用预训练网络。预训练网络(pretrained network)是一个保存好的网络，之前已经在大型数据集上训练好。如果原始数据集足够大且足够通用，那么预训练网络学到的特征可用于各种不同的计算机视觉问题，即使新问题涉及的类别和原始任务完全不同。这里将使用VGG16架构来实现迁移学习。VGG16是2014年提出来的一种卷积神经网络，由于它的简洁性和实用性，很快成为了当时最流行的卷积神经网络模型，后面出现了许多更为复杂和先进的新模型，比如inception,resnet,inception-resnet,xception等。

### 预训练模型和训练数据下载

Keras提供了包括VGG16在内的多种架构的预训练模型，下载地址如下：   
https://github.com/fchollet/deep-learning-models/releases/  

   ![](/images/images_2018/11-10_04.png)   

下载vgg16_weights_tf_dim_ordering_tf_kernels.h5和vgg16_weights_tf_dim_ordering_tf_kernels_notop.h5两个文件，并把它们拷贝放在C:\Users\<username>\.keras\models目录下。    

本文用到的猫狗分类的原始数据，可以从https://www.kaggle.com/c/dogs-vs-cats/data 下载，如果没有Kaggle帐号的话，需要注册一个，过程很简单。这个数据集包含25000张猫狗图像，每个类别各12500张。为了训练，需要创建一个新数据集，其中包含三个子集：每个类别各1000个样本的训练集，每个类别各500个样本的开发集，和每个类别各500个样本的测试集，可以写一个脚本自动完成这件事。由于每个类别的样本数相同，这是一个平衡的二分类问题，精度可作为衡量成功的指标。

### 特征提取

用于图像分类的卷积神经网络包含两部分：首先是一系列的池化层和卷积层，最后是一个密集连接分类器，第一部分叫做模型的卷积基(convolutional base)。对于卷积神经网络而言，特征提取就是取出之前训练好的网络的卷积基，在上面运行新数据，然后在输出上面训练一个新的分类器。

为什么仅重复使用卷积基？原因在于卷积基学到的表示可能更加通用，而分类器学到的表示仅包含类别的概率信息。模型中更靠近底部的层提取的是局部的、高度通用的特征图，比如边缘、颜色和纹理，而更靠近顶部的层提取的是更加抽象的概念，比如猫耳朵或狗眼睛。

首先，将VGG16卷积基实例化：  

    from keras.applications.vgg16 import VGG16

    base_model = VGG16(input_shape=(224,224,3),
            weights='imagenet',
            include_top=False)


这里向构造函数传入了三个参数：   
 - input_shape 是输入到网络中的图像张量的形状。VGG16默认的形状是(224,224,3)。  
 - weights 指定模型初始化的权重参数，我们使用了原来的参数。
 - include_top 指定模型最后是否包含密集连接分类器。因为我们要用自己的分类器，所以这里不需要包含它。

接下来，我们将扩展上述模型，在顶部添加一个密集连接分类器：  

    from keras.layers import Flatten, Dense
    from keras.models import Model

    x = base_model.output
    x = Flatten()(x)
    x = Dense(256, activation = 'relu')(x)
    predictions = Dense(2, activation = 'softmax')(x)
    model = Model(inputs=base_model.input, outputs=predictions)  

在编译和训练模型之前，一定要冻结卷积基。冻结一个或多个层是指在训练过程中保持其权重不变。因为如果不这么做，卷积基之前学到的表示将会在训练过程中被修改。冻结方法如下：  

    for layer in base_model.layers:
        layer.trainable = False

为了让修改生效，需要编译模型

    from keras.optimizers import Adam

    model.compile(loss='categorical_crossentropy',
        optimizer=Adam(lr=2e-5),
        metrics=['accuracy'])  

我们这里使用数据增强的特征提取： 

    from keras.preprocessing.image import ImageDataGenerator

    rain_datagen = ImageDataGenerator(
        rescale=1./255,
        rotation_range=40,
        width_shift_range=0.2,
        height_shift_range=0.2,
        shear_range=0.2,
        zoom_range=0.2,
        horizontal_flip=True,
        fill_mode='nearest')

    validation_datagen = ImageDataGenerator(
        rescale=1./255)


    train_generator = train_datagen.flow_from_directory(
        train_directory,
        target_size=target_size,
        batch_size=batch_size,
        class_mode='categorical')

    validation_generator = validation_datagen.flow_from_directory(
        validation_directory,
        target_size=target_size,
        batch_size=batch_size,
        class_mode='categorical') 

进行端到端的训练： 

    history = model.fit_generator(
        train_generator,
        steps_per_epoch=steps_per_epoch,
        epochs=feature_extraction_epoch,
        validation_data=validation_generator,
        validation_steps=validation_steps,
        callbacks=callbacks)
 
### 模型微调

模型微调(fine-tuning)与特征提取互为补充。对于用于特征提取的冻结的模型基，微调是指将其顶部的几层解冻，并将解冻的几层和新增加的部分联合训练。之所以叫微调，是因为它只是略微调整了模型中更加抽象的表示，以便让这些表示和新问题更加相关。我们将微调最后三个卷积层。

    GAP_LAYER = 15 #block5_conv1

    for layer in model.layers[:GAP_LAYER]:
        layer.trainable = False
    for layer in model.layers[GAP_LAYER:]:
        layer.trainable = True

如何找到想要的GAP_LAYER，这里提供一个trick，通过以下代码可以查看模型的layer number以及对应的layer name: 

    model = VGG16(weights='imagenet')
    for i, layer in enumerate(model.layers):
        print(i, layer.name) 

执行结果如下： 

	0 input_1
	1 block1_conv1
	2 block1_conv2
	3 block1_pool
	4 block2_conv1
	5 block2_conv2
	6 block2_pool
	7 block3_conv1
	8 block3_conv2
	9 block3_conv3
	10 block3_pool
	11 block4_conv1
	12 block4_conv2
	13 block4_conv3
	14 block4_pool
	15 block5_conv1
	16 block5_conv2
	17 block5_conv3
	18 block5_pool
	19 flatten
	20 fc1
	21 fc2
	22 predictions

我们很清楚的看到最后三个卷积层是block5_conv1,block5_conv2和block5_conv3，其中block5_conv1的layer number是15，即是我们要找的GAP_LAYER。

为什么不微调更多层？一个原因是训练的参数越多，过拟合的风险越大，另一个原因是微调越靠近底部的层，得到的回报越少，顶部的层编码的是更专业化的特征，更有用。

接下来就可以编译并微调网络了，这个部分我们使用了更小的学习率，原因在于我们希望微调的三层表示，其变化范围不要太大，太大的权重更新会破坏这些表示。

    model.compile(loss='categorical_crossentropy',
        optimizer=Adam(lr=1e-5),
        metrics=['accuracy'])

    history = model.fit_generator(
        train_generator,
        steps_per_epoch=steps_per_epoch,
        epochs=fine_tune_epoch,
        validation_data=validation_generator,
        validation_steps=validation_steps,
        callbacks = callbacks)

### 绘制结果  

我们可以绘制训练期间的损失曲线和精度曲线：  

    import matplotlib.pyplot as plt

    acc = history.history['acc']
    val_acc = history.history['val_acc']
    loss = history.history['loss']
    val_loss = history.history['val_loss']

    epochs = range(1,len(acc)+1)

    plt.plot(epochs, acc, 'bo', label = "Training acc")
    plt.plot(epochs, val_acc, 'b', label = "Validation acc")
    plt.title('Training and validation accuracy')
    plt.legend()
    plt.figure()

    plt.plot(epochs, loss, 'bo', label = "Training loss")
    plt.plot(epochs, val_loss, 'b', label = "Validation loss")
    plt.title('Training and validation loss')
    plt.legend()
    plt.show()

## 总结 

进行迁移学习的步骤如下：  

- 在已经训练好的基网络上添加自定义网络
- 冻结基网络
- 训练所添加的部分
- 解冻基网络的一些层
- 联合训练解冻的这些层和添加的部分



参考资料：  
1、NIPS，全称神经信息处理系统大会(Conference and Workshop on Neural Information Processing Systems)，是一个关于机器学习和计算神经科学的国际会议。该会议固定在每年的12月举行,由NIPS基金会主办。NIPS是机器学习领域的顶级会议。在中国计算机学会的国际学术会议排名中，NIPS为人工智能领域的A类会议。   
2、https://www.sohu.com/a/134792245_717210    
3、http://vc.cs.nthu.edu.tw/home/paper/codfiles/melu/201604250548/VGG.pdf  
4、Python深度学习
 
   




