# ShuffleNetV1
##### 翻译：http://tongtianta.site/paper/771

### 概要

引入两种新操作：

1.逐点组卷积

群组卷积首次提出是在AlexNet中将模型分布到两个GPU上

2.通道整合



Xception ResNeXt网络的小型化使得这种先进的基础架构变得不那么有效，其主要原因是过大的1x1Conv计算复杂度

（理解：1x1Conv不能承受高计算复杂度，只能满足低计算复杂度的计算需求）

MobileNet将可分离卷积引入到网络中，本论文在此基础上在引入了组卷积



### 架构

1x1 Conv | 3x3 DWConv | 1x1 Conv  ---->  1x1 GConv | Channel Shuffle | 3x3 DWConv | 1x1 GConv



# ShuffleNetV2
##### 翻译：http://tongtianta.site/paper/2905

### 概要

从实际生产环境出发：

以往的移动端的CNN设计在考虑计算节省时都直接致力于优化整体网络计算所需的Flops。

但实际上一个网络模型的训练或推理过程Flops等计算只是其时间的一部分，其它像内存读写/外部数据IO操作等都会占不小比例的时间。

为实际生产考虑，不应只限于去片面追求理论Flops的减少，更应该去看所设计的网络实际部署在不同类型芯片上时却具有的实际时间消耗。

针对嵌入式指标对ShuffleNetV1做修改



分析ＭobileNet ShuffeNetV1在GPU ARM上每个阶段的耗时，总结出

#####　高效CNN网络设计的四个准则

* 当输入，输出channels数目相同时，conv计算所需的MAC(memory access cost)最为节省

* 过多的Group conv会加大MAC开销

* 网络结构整体的碎片化会减少其可并行优化的程序

* Element-wise(对应元素逐个相乘)操作是一种典型memory access密集操作，所以它对整个网络计算所需时间影响较大，不可轻视



ShuffleNetV1违反规则：

首先它使用了bottleneck 1x1 group conv与module最后的1x1 group conv pointwise模块，使得input channels数目与output channels数目差别较大，违反了上述规则一与规则二；

其次由于它整体网络结构中过多的group conv操作的使用从而违反了上述规则三；

最后类似于Residual模块中的大量Element-wise sum的使用则进而违反了上述规则四。

ShuffleNetV2的改进：

ShuffleNet v2中弃用了1x1的group convolution操作，而直接使用了input/output channels数目相同的1x1普通conv。

它更是提出了一种ChannelSplit新的类型操作，将module的输入channels分为两部分，一部分直接向下传递，另外一部分则进行真正的向后计算。

到了module的末尾，直接将两分支上的output channels数目级连起来，从而规避了原来ShuffleNet v1中Element-wise sum的操作。

然后再对最终输出的output feature maps进行RandomShuffle操作，从而使得各channels之间的信息相互交通。



【我的思考】
从实际出发，分析每一个实际问题，做实验针对性update