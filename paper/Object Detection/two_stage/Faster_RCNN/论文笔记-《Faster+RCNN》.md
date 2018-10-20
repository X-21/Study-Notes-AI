# Faster RCNN
#翻译： 	
http://noahsnail.com/2018/01/03/2018-01-03-Faster%20R-CNN%E8%AE%BA%E6%96%87%E7%BF%BB%E8%AF%91%E2%80%94%E2%80%94%E4%B8%AD%E6%96%87%E7%89%88/
### 概要

SPP-Net、Fast-RCNN已经降低了检测网络的运行时间，而区域提出则成为了一个瓶颈。

本文提出了RPN网络（Region Proposal Network）用于替代Fast-RCNN中的SS。且RPN与Fast-RCNN共享一个检测网络——相当于引入了注意力机制。

在VGG中，达到了5fps的检测速度（包含所有运算），在COCO数据集中，每张图像只有300个proposal（对比SS有约2K）

对于region proposal而言，如果使用SS，那么每张图像需要在CPU上占用2S。如果使用EdgeBoxes，每张图像需要在CPU上占用0.2S，扔与整个detection network消耗了相同的时间。

根据作者的观察，用于object detection的convolutional network同样能用于region proposal。作者在CNNs的末尾添加了一些额外的卷积层（FCN，fully convolutional network）来同时进行regress region bounds以及对象评分。基于这种结构，RPN网络可以进行端到端的训练。

为了有效预测各种尺寸、长宽比的region proposal有以下三种方案：

+ multi scaled images
+ multi filter sizes
+ multi references



【我的思考】

+ 根据代码来看，anchor仅仅是一种虚拟的东西。regr网路仅仅是1x1卷积，然后输出anchor_num * 4 个channel。相当于所有的信息都来源于1个feature（包含多个channel，此处的1个feature并非backbone的1个feature，因为中间还接了一个3x3卷积，可能是用于扩大感受野），而1个feature的感受野是固定的一个矩形。相当于在input image上做固定尺寸的滑动窗口。然后每个窗口对应的1个feature，进行anchor_num次bbox回归。
+ 基于上述实现，一个feature单元，最多可以发现anchor_num个class，对于密集型的场景难以有效的发现
+ 由于RPN之后接了NMS（Non-maximum  suppression），因此密集型的对象，例如多个人挤在一起的场景，难以有效的分割。



### 训练方式

+ fine tuning RPN
+ 保持RPN不变（共享的CNNs照常训练），fine tuning object detection network



###构造RPN训练集

+ 与groud truth之间IOU最大的anchor置为positive
+ 与groud truth之间IOU超过0.7的anchor置为positive
+ ……IOU小于0.3的anchor置为negative
+ 其余样本不参与学习

【我的理解】

+ 第一、第二条规则确保了groud truth至少能够对应到一个anchor，即能够参与训练

+ 第三条规则只将哪些与所有groud truch box都相差较大的anchor置为negative，这样确保了有足够的positive，因为RPN网络主要追求的是recall，因为RPN如果漏了某个region，那么就一定会错误，而如果多了，后面还有一级detection网络，可以进行修正。


### anchor数量测试

将anchor从1增大到9，mAP随之增加。

![img](http://upload-images.jianshu.io/upload_images/3232548-826524060db73235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 



### RPN的理解

知乎上的一个回答：

anchor 在图像中对应的尺寸和长宽比例是各种各样的，有的甚至还超过了滑动窗口的 3x3 输入的感受野的范围。如果说一块 feature map 通过网络的学习，只关注感受野内某一块区域的信息，这个是合理的，但要说它同时来自 9 个不同的区域，这显然有点魔幻的感觉，更不用说有的区域根本就超出了它感受野的范围。

anchor 让网络学习到的是一种推断的能力。网络不会认为它拿到的这一小块 feature map 具有七十二变的能力，能同时从 9 种不同的 anchor 区域得到。拥有 anchor 的 rpn 做的事情是它已知图像中的某一部分的 feature（也就是滑动窗口的输入），判断 anchor 是物体的概率。anchor 可能比感受野大，也可能比感受野小，如果 anchor 比感受野大，就相当于只看到了我关心的区域（anchor）的一部分（感受野），通过部分判断整体，如果比感受野小，那就是我知道比我关心的区域更大的区域的信息，判断其中我关心的区域是不是物体。



我的理解：

不同尺寸的anchor的感受野其实是一样的，例如都看到了一个人的下半身。而anchor学习到的是一种推断能力。例如刚才的例子，从一个人的下半身，可以大概率推断出一个竖立的长方形anchor。而不会推断出一个横躺的长方形。并且，根据下半身在感受野中的位置，偏左偏右，还能对该竖立的长方形anchor进行一点微调，得到最终的一个region proposal。