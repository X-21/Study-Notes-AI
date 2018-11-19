# MobileNets
#翻译：https://blog.csdn.net/hrsstudy/article/details/78677236

### 概要

虽然CNN的设计趋势是设计更深、更复杂的网络来获取更高的精度
然而在实际应用中，由于模型尺寸和速度的约束，这种以模型大小和速度为代价来提高模型精度的改进的需求并不大
提出MobileNet的网络结构，其中包含width multiplier、resolution multiplier两个辅助构建更小、更高效MobileNets的超参数
让模型构建者可以根据模型实际应用场景的约束，在构建模型时选择合适的模型尺寸，从而对模型的运行速度和精度进行有效地权衡。

### 亮点

通过Depthwise Separable Convolution来压缩模型
https://blog.csdn.net/u012426298/article/details/80998547
一个DK×DK×M×N的标准卷积层可以因式分解为一个DK×DK×1×M的depthwise卷积层和一个1×1×M×N的pointwise卷积层。
一个标准的卷积层把滤波和融合输出放在一步进行。而depthwise separable卷积层把这个实现分为2层，一层做滤波一层做融合。这种因式分解可以有效的降低计算量和模型尺寸。

### 网络结构

MobileNet的depthwise separable卷积层使用3×3的卷积核，计算量只有使用相同卷积核的标准卷积层的18到19 ，同时精度只略微下降了一点。 
MobileNet在depthwise separable卷积层的depthwise卷积层和pointwise卷积层中均加入了BN和ReLU。

两个控制网络大小的超参数：
1.Width Multiplier: Thinner Models
虽然基础MobileNet结构已经非常小、延迟非常低，但是很多时候一些特殊的应用场景需要更小、更快的模型。为了能够方便的构建这些尺寸更小、计算量也更小的模型
引入了一个简单的参数width multiplier α。width multiplier α的作用是让网络的每一层均匀的变瘦。在指定width multiplier α后，对任意一层，输入通道数 M 变成αM，输出通道数N变成αN。
Width multiplier可以有效的减小计算量和参数量约α2倍
（相当于直接去较小了通道数量，这个参数就是来控制通道数量的比例）
2.Resolution Multiplier: Reduced Representation
resolution multiplier ρ被应用于输入图像，随后网络中的每层的内部表示也被减小到相同的ρ倍。实际上，我们通过设置输入的分辨率来隐式的设置ρ。
（相当于去调节了输入图像的分辨率，从而导致每一层的特征图都会出现相同比例的缩小）
resolution multiplier ρ可以有效的降低计算量为之前的ρ2倍。 


【我的思考】
目前项目的落地，特别是针对边缘计算的落地
考虑到模型要运用到设备端，对模型的压缩需要尤为重要
目前有几种常见的压缩方法，要实现项目的落地，模型压缩将是不可或缺的一部分


