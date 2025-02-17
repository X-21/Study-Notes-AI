# U-Net
##### 翻译：http://tongtianta.site/paper/1301

https://blog.csdn.net/natsuka/article/details/78565229

### 概要

U-Net网络结构画出来形似字母U,因而得名U-Net (QAQ)

U-Net基于FCN改进，利用数据增强可以对一些比较少样本的数据进行训练，特别是医学方面相关的数据．

### 架构

U-net采用了完全不同的特征融合方式：拼接，U-net采用将特征在channel维度拼接在一起，形成更厚的特征。而FCN融合时使用的对应点相加，并不形成更厚的特征。

网络由两部分组成：一个**收缩路径(contracting path)**来获取上下文信息以及一个对称的**扩张路径(expanding path)**用以精确定位。

contracting path是典型的卷积网络架构。它的架构是一种重复结构，每次重复中都有2个卷积层和一个pooling层，卷积层中卷积核大小均为3 *3，激活函数使用ReLU，两个卷积层之后是一个2 *2的步长为2的max pooling层。

每一次下采样后我们都把特征通道的数量加倍。contracting path中的每一步都首先使用反卷积(up-convolution)，每次使用反卷积都将特征通道数量减半，特征图大小加倍。

反卷积过后，将反卷积的结果与contracting path中对应步骤的特征图拼接起来。

contracting path中的特征图尺寸稍大，将其修剪过后进行拼接。对拼接后的map进行2次3 *3的卷积。

最后一层的卷积核大小为1*1，将64通道的特征图转化为特定深度（分类数量，二分类为2）的结果。网络总共23层。



### 训练

采用弹性形变的方式增强数据，让模型学习得到形变不变性．

为了预测像素边界区域的像素点，采用镜像图像的方式补齐缺失的环境像素．



【我的思考】
同FCN作为像素级分割从　０到１　的实现


