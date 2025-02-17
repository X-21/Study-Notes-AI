# MobileNetV1
翻译：http://tongtianta.site/paper/583 
资料：https://blog.csdn.net/stesha_chen/article/details/82699331 


### 概要

在现实任务中,需要满足识别任务实时进行
本论文描述了一个有效的网络架构，以便构建非常小的低延迟模型，可以轻松地匹配移动和嵌入式视觉应用的设计要求
(引入深度可分离卷积)

### 深度可分卷积

一种分解卷积的形式
深度可分卷积由两层组成：深度卷积和逐点卷积。使用深度卷积为每个输入通道（输入深度）应用单个滤波器。然后使用简单的卷积的点线卷积来创建深度层的输出的线性组合。
其代替原始卷积的方法:
3x3 Conv | BN | ReLU  -->  3x3 Depthwise Conv | BN | ReLU | 1x1 Conv | BN | ReLU

1.\[DF,DF,M\]拆成M个\[DF,DF,1\]的矩阵，每个通过\[DK,DK,1\]的卷积核得到\[DF,DF,1\]的矩阵，共是DK\*DK\*DF\*DF\*M次计算； 
2.将\[DF,DF,M\]矩阵通过N个\[1,1,M\]的核得到\[DF,DF,N\]，共M\*DF\*DF\*N次计算。 

【我的思考】
成功引入了深度可分离卷积,使得计算量很小的情况小可以达到接近大网络模型的效果



# MobileNetV2
翻译：http://tongtianta.site/paper/1783 
资料：https://blog.csdn.net/stesha_chen/article/details/82744320 
https://blog.csdn.net/u011974639/article/details/79199588 
https://blog.csdn.net/u014380165/article/details/79200958 
https://blog.csdn.net/weixin_37675458/article/details/83350022 
https://zhuanlan.zhihu.com/p/33075914 
### 概要

倒置残差和线性瓶颈
保留V1的简单性,进一步提高性能

### 线性瓶颈
ReLU会对channel数较低的张量造成较大的信息损耗
论文中给出当原始输入维度数增加到15以后再加ReLU，基本不会丢失太多的信息；但如果只把原始输入维度增加至2~5后再加ReLU，则会出现较为严重的信息丢失。
(Relu对于负的输入，输出全为零,而本来特征就已经被“压缩”，再经过Relu的话，又要“损失”一部分特征，因此这里不采用Relu)
论文执行降维的卷积层后面不会接类似于ReLU这样的非线性激活层
相对于V1的网络结构区别:
V1: 3x3 Depthwise Conv | BN | ReLU | 1x1 Conv | BN | ReLU
V2: 1x1 Conv | BN | ReLU | 3x3 Depthwise Conv | BN | ReLU | 1x1 Conv | BN | Linear
先通过了一个1x1卷积来提升通道数获得更多的特征,最后使用线性的激活函数避免低通道的特征经过relu后丢失信息

### 倒置残差
借鉴resnet中的残差结构
DWConv layer层提取得到的特征受限于输入的通道数，若是采用以往的residual block，先“压缩”，再卷积提特征，那么DWConv layer可提取得特征就太少了，因此一开始不“压缩”
MobileNetV2反其道而行，一开始先“扩张”，本文实验“扩张”倍数为6。 通常residual block里面是 “压缩”→“卷积提特征”→“扩张”，MobileNetV2就变成了 “扩张”→“卷积提特征”→ “压缩”，因此称为Inverted residuals


【我的思考】
相对V1主要两点改进:倒置残差和线性瓶颈
都是避免小网络中对特征的损失过多而做的tricks 

【思考2】
不理解： 
In supplemental materials, we show that if the input manifold can be embedded into a signiﬁcantly lower-dimensional subspace of the activation space then the ReLU transformation preserves the information while introducing the needed complexity into the set of expressible functions.
说的是在补充物料中，提到了一点： 
如果输入兴趣显著地嵌在激活空间的低维子空间里，那么ReLU变换将保留信息，同时为所需的复杂性引入一组可变达的函数。
