# Mask RCNN
翻译：https://blog.csdn.net/myGFZ/article/details/79136610
### 结构

两个关键技术点：
1.在Faster RCNN的detection网络的cls、reg分支外，新增一个mask分支（FCN）。
2.修改ROI poolomh 为ROI align(像素对齐)

掩膜分支输出维度是k*m*m的，k是类别数量，m*m是掩膜大小
对每个像素使用sigmod(阈值0.5二值化)后加二进制交叉熵损失函数，对于真实类别为k的RoI，仅在第k个掩模上计算Lmask（其他掩模输出不计入损失）
输出维度k为每个类别独立的预测，从而不产生跨类别的竞争

###训练
每个采样后的RoI上的多任务损失函数定义为： 
L = Lcls + Lbox + Lmask

###Roi Pooling 与 Roi align

![img](https://img-blog.csdn.net/20180308194456806) 

Roi pooling一共做了两次量化，导致像素偏移：

+ 通过RPN得到proposal后实际上是一个float（regress结果），但是将其映射到feature map后有一个取整操作。
+ ROi pooling的过程中，将Roi几等分，这个时候可能出现除不尽的情况，例如feature map为30x30，而Roi为7x7，这个时候就有取整

###实验
1.类相关和类无关掩模（解藕了分类和分割）
使用像素级softmax和非独立损失的方法进行比较，产生了产生类间竞争，性能大幅度下降

2.可以扩展到人体姿势估计
将关键点的位置建模为one-hot掩模，并采用Mask R-CNN来预测K个掩模，每个对应K种关键点类型之一（例如左肩、右肘）


Roi align操作：

+ proposal 映射到 roi时，保留浮点数。
+ 进行与roi pooling一样的等分规则，此时依然保留浮点数
+ 将候选区域划分为KxK个区域，然后每个区域选出固定四个位置的feature点（坐标是float），然后每一个feature点，使用最近的4个真实feature点（整数），进行双线性插值，获得估计值，最后使用这四个点做max pooling得到最终每个区域的特征值（KxK）。

我的思考：
在语义分割这一块，Mask RCNN有着重要的意义，基本上是目前进行图像分割最先进的技术（2018.10.15）
在京东风格识别上我们用了mask rcnn来做人物分割然后提取衣服颜色，可以证明此技术的可扩展意义