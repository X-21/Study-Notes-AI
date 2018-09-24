# SPP-Net
#翻译：https://blog.csdn.net/mengduanhonglou/article/details/78470682

###概述

CNNs需要固定的尺寸，大小以及比例。对于任意尺寸的输入，需要通过crop或者warp的方式将输入转化为固定的尺寸。而crop会导致图片信息的丢失，而warp则会发生形变导致预训练的模型效果下降。

CNNs固定尺寸的原因是由于最后的FC层需要固定尺寸的输入。而本文则通过在最后的convolutional layer与fc层之间添加SPP-Layer，来将任意尺寸的feature map转换为固定尺寸的feature map。

SPP已经是一种成熟的技术应用在CV领域，但是本文是首次将其用于CNNs。

###SPP的优势

+ 将变尺寸的feature map转化为固定尺寸的feature map
+ SPP使用不同大小的window，相比于使用固定window的传统pooling，具有更好的鲁棒性。
+ 能够在任意尺寸的图片下提取特征
+ SPP允许在train的过程中使用不同尺度的图片，这有助于学习到尺度不变性，减少over-fitting。本文使用每个EPOCH不同image size的方式进行训练，得到了更好的test accuracy。
+ SPP能够解决RCNN每一个region都要resize然后计算feature map这一耗时的过程。SPP方案中，仅仅需要计算一次全图的feature map，然后根据不同的region获得对应的sub feature map，最后通过SPP将不同尺寸的sub feature map转化为固定尺寸的feature map（对于比RCNN，提升了24 ~ 102倍。当使用EdgeBoxes替代RCNN中的SS后，可以做到0.5秒处理一张图像）。
+ SPP和CNNs具有正交性，能够应用在不同的CNN网络中，都能进行一定的提升。




【我的思考】总结一下SPP的特点

+ output fix size feature vector
+ multi pooling stride


容易混淆的点：

+ SPP本身仅仅是对CNNs的输出进行pooling，无论pooling的stride是多大，都不能提取多尺度特征，它需要依赖图像金字塔（图片尺寸的缩放）来获取不同尺寸的特征

###SPP-Pooling流程

+ 卷积网络的输入为(w, h, k)
+ 使用变stride卷积（无论feature map的长宽，都将其划分为指定数量的区域进行卷积，然后拼接。例如论文中实验使用了6x6、3x3、2x2、1三种划分数量。即无论feature map的尺寸，卷积之后的feature vector长度都为36 + 9 + 4 + 1。这儿划分数量为1的pooling其实就是global pooling）



### SPP用于Classification

文中为了验证SPP的对CNNs的性能改善进行了一系列的实验：

+ 使用单一尺寸+SPP：将图片crop到固定的224x224
  + 发现性能有所提升。
  + 说明SPP的多尺度特征提取这个特性能够提升网络的性能。
+ 使用多尺寸+SPP：将图片crop到224x224，然后再resize到180x180，然后利用这两种尺寸进行训练
  + 相比于实验1，性能有所提升，且收敛更快。
  + 这有可能是因为学习到了一定的尺度不变性，或者因为图片尺寸变化相当于做了augmentation。
+ 使用完整图像：将图像进行resize，使其短边长度为256（保留比例）。
  + 相比于实验1，性能有所提升，但不如实验2。
  + 完整图像保留了原图的所有信息，这在例如图像修复等领域是 具有优势的。



作者在测试时，将图片resize到了不同的尺寸，然后再使用crop（固定224） + flip等手段获得多个sub image进行预测然后取平均，最后将错误率降低了1.59%。这可以看做是SPP的优势，传统的Pooling虽然可以进行crop + flip但是没有办法进行resize。



【我的思考】多尺度训练普遍能够提升CNNs的准确率

### SPP用于Object detection

SPP改善了RCNN的重复特征计算步骤，仅仅需要计算一次。

