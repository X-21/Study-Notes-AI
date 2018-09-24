#RCNN

#翻译：https://blog.csdn.net/wangsidadehao/article/details/54237567

### 概要

在上一个十年，通过HOG、SIFT等低阶特征提取手段以及复杂的机器学习方法，物体识别和定位取得了重大的进步。但是在2010之后，进步逐渐放缓，只有模型集成、优秀算法的变体取得了较小的进步。

HOG、SIFT是半局部的趋向性直方图，大致相当于V1（灵长类的第一个复杂的视觉皮层）。而识别是有多个阶段的，且这种层级的、多阶段的特征计算过程可能是更为重要的部分。

AlexNet使用CNN在ILSVRC上取得重大突破后，如何将CNN应用到PASCAL VOC中成为了一个火热的议题。

本文通过解答两个问题，来跨域了图像分类和对象检测之间的鸿沟：

+ 如果通过深度网络来定位物体的位置
+ 如果通过少量的标注数据来训练一个大容量的模型（大量参数）



### 物体定位及分类


常用的一些候选区选择方法：

+ 回归：只适用于单对象探测
+ 滑动窗口：窗口的大小、比例固定，导致难以使用复杂的场景。
+ selective search



本文使用的方法：

+ 基于selective search算法，选择2K个region
+ 使用region进行分类训练，实现网络的fine tuning
+ 将region reshape（anisotropic image scaling）到相同的尺寸，然后输入到CNN中得到固定尺寸的feature vector
+ 通过SVM进行分类


上述的第二步中，之所以要进行reshape是因为最终的分类和bbox回归都需要接受指定尺寸的feature，而SS得到的region是不同尺寸的，因此无法直接使用region输入CNN。



我的思考：RCNN是一种三阶段的训练
其中cnn只是作为哦特征提取，作为SVM的输入
+ find region proposal
+ calc feature map
+ classification

### 解决样本不足的问题

本文发现通过在通用的大数据集（ILSVRC等大数据）上进行预训练，然后在特定领域的小数据集（PASCAL VOC)上进行fine tune是一种高效训练大容量CNN的范式。
预训练是图像级的，没有定位标注。

本文所做的实验表明，fine tune提升了8%的mAP，并最终达到了63%。而基于HOG的高度调优DPM模型则为33%。



### 瓶颈

+ test阶段，每个region（最多2K）都要重复调用CNNs计算feature，非常耗时