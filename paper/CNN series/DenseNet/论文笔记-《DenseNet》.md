# DenseNet
#翻译：https://blog.csdn.net/u011974639/article/details/78290448

### 概要

以往提升神经网络的性能主要从深度和宽度着手
而本论文从feature入手，将feature运用到极致
作者提出神经网络并不一定就是递进层级架构（网络某一层不仅仅依赖上一层输出，可以依赖更早的学习特征）
改善了ResNet的冗余性，因为ResNet实际每一层学习的特征很少，即使丢掉几层也不会影响其性能
实现了更深的网络，更小的计算开销，更强的网络性能

其设计理念为：
让每一层都直接与前面的层相连，实现特征的重复利用
同时将网络设计的很窄，即每一层只学习到很少的特征，达到降低冗余性的目的

### 亮点

1.防止网络深度加深而退化，继承ResNet的思想，但新的核心思想是：
保证网络中层与层之间最大程度的信息传输的前提下，直接将所有层连接起来（l*(l+1)/2）
对于网络的任意一层，该层前面所有层的feature map都是这层的输入，该层的feature map是后面所有层的输入
（这样的一个全连接网络称为一个dense block）

2.为了进一步简化网络，防止参数过多，使用了几个技巧
1)使用1x1来降维，类似ResNet的做法，dense block内的层连接方式为：
BN→ReLU→Conv(1×1)→BN→ReLU→Conv(3×3)
2)dense block间通过transition layers连接
BN→Conv(1×1)→Avg  Pool(2×2)
在transition layers加上参数θ来控制输出的feature maps数量

### 网络结构

+ dense block内层连接(Composite function)
BN→ReLU→Conv(1×1)→BN→ReLU→Conv(3×3)

+ dense block间连接
BN→Conv(1×1)→Avg  Pool(2×2)

+ Growth rate
如果一个H(l)输出k个feature maps，那么l(th)层有k0+k×(l−1)个feature maps输入,k0是输入层的通道数。
k太大会导致参数过多，于是将k作为一个超参数

+ 与ResNet在合并上的区别
ResNet直接将输出加到输出上
DenseNet做的是拼接操作，保留了浅层的原始特性

+ 与ResNet随机连接的相似性
ResNet的随机连接也让一些浅层原始输出直接传递到了下一层，和DenseNet的思想有相似之处

### 实验

+ L和k的增加，模型的效果会更好
+ 相比ResNet达到同一错误率时，参数复杂度明显要低很多


【我的思考】
可以理解为对ResNet的升级
从一个新的角度feature来提高网络性能，强化了特征传播
特征利用是的前面的特征直接使用（通过拼接的方式直接保留原始输出），后面在学习就不会再去学习前面学到的特征，虽然学到的不多，但是是新的特征

