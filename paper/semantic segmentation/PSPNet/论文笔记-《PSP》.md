PSPNet

##### 翻译：http://tongtianta.site/paper/2081

https://blog.csdn.net/u011974639/article/details/78985130

### 概要

提出的金字塔池化模块( pyramid pooling module)能够**聚合不同区域的上下文信息**,从而提高获取全局信息的能力。

场景解析(Scene Parsing)的难度与场景的标签密切相关

* **Mismatched Relationship**：上下文关系匹配对理解复杂场景很重要
* **Confusion Categories**： 许多标签之间存在关联，可以通过标签之间的关系弥补
* **Inconspicuous Classes**：模型可能会忽略小的东西，而大的东西可能会超过FCN接收范围，从而导致不连续的预测

论文作出以下贡献：

* 提出了一个金字塔场景解析网络，能够将难解析的场景信息特征嵌入基于FCN预测框架中
* 在基于深度监督损失ResNet上制定有效的优化策略
* 构建了一个实用的系统，用于场景解析和语义分割，并包含了实施细节







### 网络架构
Pyramid Pooling Module(PSP模块)

* **用全局平均池化处理**。但这在某些数据集上，可能会失去空间关系并导致模糊。
* **由金字塔池化产生不同层次的特征最后被平滑的连接成一个FC层做分类**。这样可以去除CNN固定大小的图像分类约束，减少不同区域之间的信息损失。



aux loss

取resnet layer3后的输出做一个额外loss，可以加速收敛



###### 我的思考

分析FPN的缺点，针对性优化，强化了网络场景理解能力


