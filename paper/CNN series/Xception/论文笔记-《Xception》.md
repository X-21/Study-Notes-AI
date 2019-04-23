# Xception
翻译：http://tongtianta.site/paper/579 
资料：https://www.jianshu.com/p/4708a09c4352 

### 概要

分析Inception模块作为卷积特征提取器是如何工作的？和常规卷积有何不同？设计策略是什么？

* Inception的多尺寸卷积：

核心思想是使用多尺寸卷积核去观察输入数据，由于景象远近不同其物体大小也会有所不同，那么不同尺寸的卷积核观察的特征就会有不同的效果

(增加了网络的宽度，同时也提高了对于不同尺寸的适应程度)

* Pointwise Convolution

宽度增加后参数量就上去了，于是使用1x1卷积来做通道降维，减小计算量

* 卷积核替换

使用两个3x3卷积代替5x5，vgg也是这个操作．减小计算量，增加深度

除了规整的卷积核还有3x3=3x1 + 1x3的分解版本，再深度较深的情况下该版本效果更好

* Depthwise Separable Conv

对一个深度图进行卷积后再融合， Depthwise Conv + Pointwise Conv，大大减少了参数量

Inception中的可分离卷积还只是引入了这种思想，没有做到mobileNet这样的尽心设计的可分离卷积

于是论文通过一步一步的演化（将分支中1x1卷积统一拿出来），就得到了mobile中用到的可分离卷积

Xception提出了可分离卷积并且引入resnet的思想而构成的网络



【我的思考】
Xception充分分析Inception的网络结构，遵循演化趋势，找到优化点

