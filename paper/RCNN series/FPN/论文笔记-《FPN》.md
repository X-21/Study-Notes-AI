# FPN

### 概要

在传统的object detection领域中，特征金字塔是一种基本组件，用于实现不同尺度目标的检测。但是在最近的object detector中（Fast RCNN、Faster RCNN），均为使用特征金字塔，因为它的计算、内存密集特性（消耗大量的计算资源）。

本文提出了一种具有横向连接的自顶向下的结构，利用了CNNs内在的多尺度、金字塔分级，来实现了低成本特征金字塔，称之为FPN（feature pyramid network）

在一个基本的Faster RCNN系统中仅添加了FCN，便达到了state-of-the-art效果。



### 多种特征金字塔结构对比

![img](https://upload-images.jianshu.io/upload_images/3232548-79414c80444765b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



a） 特征化图像金字塔：通过图片的缩放来获得不同尺度的特征

+ 由于成倍的计算开销，因此可行性不高，且大尺寸的图片会占用很大的内存，而如果仅仅在测试时使用又会造成训练、测试不一致，由于这些原因，在Fast、Faster RCNN中并未使用特征化图像金字塔。

b）单一feature map：只使用CNNs的最高层级的特征进行预测

+ Fast、Faster RCNN使用了这种方式

c）金字塔特征层级：利用CNNs内在的层级性，分离的使用不同层级的特征

+ 不同层级的feature的语义具有较大的差异，尤其是高分辨（低层）的特征普遍是低级特征，会损害物体检测的效果
+ SSD虽然采用了这种方式，但是为了防止使用低层次特征，因此在CNNs后新增了几个卷积层，用于特征提取。

d）特征金字塔：不仅使用CNNs内在的层级性，并对不同层级进行自顶向下的级联



在手工特征设计阶段，特征金字塔的被大量应用，在最近的比赛中，所有排名靠前的队伍都使用特征金字塔（ResNet等）。



![Figure 2](https://upload-images.jianshu.io/upload_images/3232548-e1ee021eac1b88be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在最近的一些论文中，提到了上方的结构，采用自顶向下与横向连接，但是只预测一次。而本文提到的方法是分别进行预测。



### 算法

现有的CNNs算法通常有多个阶段，每个阶段内部的feature map尺寸是相同（same pooling），而阶段间的尺寸通常是两倍关系（pooling stride = 2）。

FPN网络选取每一个阶段的最后一层的输出，考虑到内存的问题，本文没有选取第一阶段。

特征自顶向下流动时，由于尺寸在变大，因此对上层特征进行上采样并与横向特征进行求和。

![Figure 3](https://upload-images.jianshu.io/upload_images/3232548-a51d06a2a94bfec4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在得到特征集合 $\{C_2, C_3, C_4, C_5, C_6\}$ 后，对其进行 3x3 卷积，去除上采样导致的混叠效应得到最终的特征集合

$\{P_2, P_3, P_4, P_5, P_6\}$ 



得到特征集合之后，将其分别输入后续的分类器和回归器（分类器、回归器，为所有层次特征值所共享）。



【我的思考】由图中的可以看出，越低的层次，融合了越多层次的特征。

### FPN与RPN

在原始的RPN中，有多尺度、多比例的anchor，而使用FPN与RPN结合后，对其进行了一定的改动。针对每个层级的不同feature map， $\{P_2, P_3, P_4, P_5, P_6\}$ 都有不同的anchor尺寸分别为$\{32^2, 64^2, 128^2, 256^2, 512^2\}$ ，比例集合与原始RPN中保持一致。这是

【我的思考】由于不同层次特征的感受野不同，实际上这些anchor的真实感受野是相同。



### FPN与Faster RCNN

对于一个特定bbox，根据其长宽，可以决定在哪一层选择其ROI。

$k=⌊k0+log_2(\sqrt {wh}/224)⌋$  

在本文中，k0 = 5

【我的思考】对象很小时，那么k < k0，即选择更低层次的特征。对象很大时，k > k0选择更高层次的特征