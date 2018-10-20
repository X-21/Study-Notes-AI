#RetinaNet

#翻译：https://blog.csdn.net/mieleizhi0522/article/details/80228990

### 概要
在yolo v1和ssd之后提出，它能够匹配之前的one-stage detector的速度，并且优于已有的state-of-the-art的two-stage detector
分析现有的one-stage之所有达不到twoi-stage的性能主要原因是因为类别不均衡，为了解决这个问题论文提出了focal loss

###focal loss
Focal loss是通过降低inliers（easy examples）的权值，这样它们对总的loss的贡献很小（即使它的值很大）
存在两个超参数


### 架构
RetinaNet：
前段网络就是就是一个FPN，反向每一层后都做出预测，相当于后再接两个子网络，一个做框回归一个做分类
同样使用anchor的机制，只是在多尺寸上，因为FPN反向时本来就是多尺寸的，所以只在方向每一层上做不同长宽比和不用形状


### 思考
主要贡献就在于focal loss，在标签不均衡的情况下效果是不错的，在比赛实践中也确实如此
在项目中遇到不均衡问题，focal loss应该作为重要实验对象