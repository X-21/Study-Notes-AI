# Mask RCNN
翻译：https://blog.csdn.net/myGFZ/article/details/79136610
### 结构

在Faster RCNN的detection网络的cls、reg分支外，新增一个mask分支。



###Roi Pooling 与 Roi align

![img](https://img-blog.csdn.net/20180308194456806) 

Roi pooling一共做了两次量化，导致像素偏移：

+ 通过RPN得到proposal后实际上是一个float（regress结果），但是将其映射到feature map后有一个取整操作。
+ ROi pooling的过程中，将Roi几等分，这个时候可能出现除不尽的情况，例如feature map为30x30，而Roi为7x7，这个时候就有取整



Roi align操作：

+ proposal 映射到 roi时，保留浮点数。
+ 进行与roi pooling一样的等分规则，此时依然保留浮点数
+ 将候选区域划分为KxK个区域，然后每个区域选出固定四个位置的feature点（坐标是float），然后每一个feature点，使用最近的4个真实feature点（整数），进行双线性插值，获得估计值，最后使用这四个点做max pooling得到最终每个区域的特征值（KxK）。