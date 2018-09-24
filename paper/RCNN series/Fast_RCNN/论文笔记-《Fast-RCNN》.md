# Fast-RCNN
#翻译：https://blog.csdn.net/xiaqunfeng123/article/details/78716060
###概述

object detection的难点：

+ 找到图片中大量的候选区
+ 精调候选区



###RCNN的不足

+ 训练过程是多阶段的（从而导致每个阶段是独立的，不能直接联系，及特征提取，spp池化，SVM，框回归相对独立）
  + 在proposal上进行fine-tuning
  + 使用SVM进行分类
  + bound box regression
+ 训练非常耗时
  + 从proposal中提取了特征后要保存到磁盘（所有图片），然后再进行SVM。对于VGG16，PASCAL VOC 2007数据集（5K）需要耗费2.5GPU-Day以及上百G的磁盘空间。
+ 对象检测非常耗时
  + 要对每个proposal进行特征提取，对于VGG16， 每张图片需要花费47s（GPU）

上述测试均使用Navidia K40

RCNN慢的原因主要是大量的特征冗余计算，SPP net对其进行改进后，测试时速度可以提升10~100倍，训练时速度可以提升3倍。



###SPP-Net的不足

+ 和RCNN一样，训练是多阶段的
+ 训练期间耗时
+ SPP-Net微调算法的无法更新SPP以前的卷积层的参数（在SPP-Net的论文中没有找到fine-tuning的细节，可能是不同版本的论文。根据其他的资料来看SPP-Net并非不能训练SPP之前的卷积层，而是训练难度较大）

spp-net与Fast-RCNN在fine-tuning上的不同：

+ SPP-Net中fine-tuning的样本是来自所有图像的所有RoI打散后均匀采样的，即RoI-centric sampling，这就导致SGD的每个batch的样本来自不同的图像，需要同时计算和存储这些图像的Feature Map，过程变得expensive.  


+ Fast R-CNN采用分层采样思想，先采样出N张图像（image-centric sampling），在这N张图像中再采样出R个RoI，具体到实际中，N=2,R=128，同一图像的RoI共享计算和内存，也就是只用计算和存储2张图像，消耗就大大减少了



### FAST RCNN的优点

+ 比RCNN、SPP-Net更高的精度
+ single-stage训练，使用multi-task loss
+ 可以在训练过程中更新所有的layer
+ 无需存储feature到磁盘





### FAST RCNN流程

+ 通过SS获得region proposal
+ 将整张图片输入CNNs获得feature map
+ 通过region proposal获得Roi（region of interest，即region proposal对应的feature map）
+ 通过Roi pooling将不同尺寸的Roi变为固定尺寸的feature map（相当于简化版的SPP，Roi pooling采用固定的windows size，例如将Roi划分为7x7，然后做Max Pooling）



对预训练网络的改造：

+ 将最后的max pooling改为roi pooling
+ 将最后的softmax改为两个layer，分别做分类和bbox回归
+ 网络输入改为接受image和roi（通过ss获得proposal，然后映射为roi）



实际上Faste RCNN可以看作是两阶段的训练（Fast RCNN论文中描述的one-stage是只将整个过程融入到了一个算法中，可以实现反向传播，进行一次性训练。而不是RCNN那样使用CNNs+SVM，分开训练）。

+ 第一阶段输入iamge，获得feature map，并通过SS与feature map获得Rois。
+ 第二阶段，预测单个Rois的class，并作bbox回归。



### Loss函数

Fast Rcnn是一种Multi-task算法，loss函数如下：

$L(p,u,t^u, v) = L_{cls}(p, u) + \lambda [u \ge 1 ] L_{loc}(t^u, v)$

$L_{cls} = -ulogp$

$L_{loc}(t^u, v) = \sum_{i \in \{x, y, w, h\}} smooth_{L_1} (t^u_i, v_i)$

$$smooth_{L_1} = \begin {cases} 0.5x^2\ \ if |x| < 1\\ |x| - 0.5 \ \  otherwise,\end{cases}$$



+ p是Classifier输出，u是真实类别。$t^u$ 是  Regressor的输出，v是真实的bbox。
+ $L_{cls}$ 是log loss
+ $\lambda$ 是二者的权重系数，本文的所有实验设置为1
+ $smooth_{L_1}$ 比起RCNN和SPP-Net中使用的$L_2$ 损失而言，不那么敏感。更易于训练，不容易出现梯度爆炸（从公式可以看出，当x很大时，loss为线性增长。至于为什么有一个0.5，不太清楚？）




### Multi-task 有用吗？

作者做了实验

+ 仅仅使用cls loss 训练
+ 使用cls loss 和 loc loss
+ 使用s、m、l三种规模的网络进行测试

使用cls + loc loss进行训练后，单看cls正确率，均提升了0.8 + 1.1个百分点。说明多任务训练是有积极效果的。



### 单尺度与多尺度训练

根据作者的实验，多尺度能够提升越1%，而使用单尺度训练+较深的网络可以大幅度的提升性能。

我的想法：网络变深了之后，多尺度的训练代价非常大。在这个trade off中，优先选择更深的网络。



### SVM与Softmax对比

根据作者的测试，使用softmax之后效果有较小幅度的提升。



### 候选框数量

mAP随着候选框的数量增加而略微上升，然后下降。因此过多的候选框反而有害。