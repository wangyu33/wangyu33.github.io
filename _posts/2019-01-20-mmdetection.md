---
layout: post
title: 我mmdetection 商汤框架
categories: Blog
description: 我的 2018 年全年盘点
keywords: 2018, 计划, 总结
---



# mmdetection 商汤框架

backbone：Resnet mobilenet darknet

neck：FPN，FAFPN

head：bbox predict， region proposal network

roi extractor：roi pooling roi align

loss：focalloss，l1loss， GHMLoss

## MobileNet

MobileNet v1在VGG基础之上使用深度可分离卷积，分组卷积数量等于对应的通道数

MobileNet v2在解决了v1中存在一些卷积核参数未训练到的情况，使用先上采样，再深度可分离卷积，在下采样的方式，并外加一个残差连接的方式，下采样为线性层。

![image-20201028102700757](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20201028102700757.png)



mobilenet v3使用hard-swish进行激活

## FPN

Feature Pyramid Networks for Object Detection（FPN）特征金子塔，类似于Unet结构，直接插值放大。

![image-20201028104032548](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20201028104032548.png)

## PAFPN

![image-20201028104632460](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20201028104632460.png)

提出双向融合的特征融合金子塔



## RoI Align



## one stage与two stage

Anchor-based主要分成两大派，一派叫做One-Stage，一派叫做Tow-Stage。两派相辅相成，又各有特色。而其中先开始进行目标检测的一派为Tow-stage。他的思想为粗定位和细分类，简单来说先从图像中找到可以存在目标的位置，然后再对这些位置进行分类。该派的发展具有非常好的传承性，基本的路线为：**R-CNN->SPP Net->Fast R-CNN->Faster R-CNN->FPN->Musk R-CNN。**依靠这个路线，Tow-stage不断完善自己的精度速度，甚至最后将目标检测升级到了实例分割，这一派牛就牛在一个’准’字上。

另一派one-stage便是以‘快’成名，他没有粗筛选的过程，直接就是定位和分类，因此也让他的速度使two-stage很难达到。而该派的主要代表工作便是**yolo v1-v4(v5貌似也有啦!)以及SSD为**代表的一众神作



anchor free  方法**CornerNet，CenterNet 和CornerNet-Lite**  **Fcos**  **yolo**

two-stage比one-stage多出region proposal环节，提取前景的大概位置。





## anchor free

**DenseBox（2015）**

不需要进行region proposal

1. 首先使用图像金字塔（这个思想后来演变成特征金字塔FPN）
2. 经过一系列卷积和池化后，再进行上采样使特征图变大（**用于检测更多的目标**），再经过一些卷积得到最终的输出
3. 把输出的特征图转换成边框，再通过NMS和阈值进行输出



Yolo3



## RetinaNet(Focal Loss)

![image-20201028172253298](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20201028172253298.png)

Focal loss主要是为了解决one-stage目标检测中正负样本比例严重失衡的问题。

one-stage目标检测识别准确率低的原因主要是由类别失衡构成的

**negative example过多造成它的loss太大，以至于把positive的loss都淹没掉了**

解决易分类样本和难分类样本之间的均衡问题，negative example不在前景和背景的过渡区域上为易分类样本

单个example的loss很小，反向计算时梯度小



Focal loss是在交叉熵损失函数基础上进行的修改，首先回顾二分类交叉上损失：

![image-20201028165350118](C:\Users\wangyu\AppData\Roaming\Typora\typora-user-images\image-20201028165350118.png)

![[公式]](https://www.zhihu.com/equation?tex=FL%3D%5Cleft%5C%7B+%5Cbegin%7Baligned%7D+-%281-p%29%5E%5Cgamma+log%28p%29++%2C%5Cquad+if%5Cquad++y%3D1%5C%5C+%5C%5C+-p%5E%5Cgamma+log%281-p%29%2C%5Cquad+if%5Cquad++y%3D0+%5Cend%7Baligned%7D+%5Cright.)

![[公式]](https://www.zhihu.com/equation?tex=FL%3D%5Cleft%5C%7B+%5Cbegin%7Baligned%7D+-%5Calpha+%281-p%29%5E%5Cgamma+log%28p%29++%2C%5Cquad+if%5Cquad++target%3D1%5C%5C++-%281-%5Calpha%29+p%5E%5Cgamma+log%281-p%29%2C%5Cquad+if%5Cquad++target%3D0+%5Cend%7Baligned%7D+%5Cright.)

**把高置信度(p)样本的损失降低**

![[公式]](https://www.zhihu.com/equation?tex=p_t) 是不同类别的分类概率， ![[公式]](https://www.zhihu.com/equation?tex=%5Cgamma) 是个大于0的值， ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha_t) 是个[0，1]间的小数,![[公式]](https://www.zhihu.com/equation?tex=%5Calpha_t+) 用于调节positive和negative的比例，前景类别使用 ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha_t) 时，对应的背景类别使用 ![[公式]](https://www.zhihu.com/equation?tex=1-%5Calpha_t) ， ![[公式]](https://www.zhihu.com/equation?tex=%5Cgamma) 和 ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha_t) 都是固定值，不参与训练。来降低easy example对loss的贡献

Focal loss存在的问题

让模型过多关注那些特别难分的样本肯定是存在问题的，样本中有**离群点（outliers）**，可能模型已经收敛了但是这些离群点还是会被判断错误，让模型去关注这样的样本，怎么可能是最好的呢？

![img](https://pic2.zhimg.com/80/v2-662e8862051dc3e9cc15d15ab8852bcd_720w.jpg)

**其次**， ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha) 与 ![[公式]](https://www.zhihu.com/equation?tex=%5Cgamma) 的取值全凭实验得出，且 ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha+++++) 和 ![[公式]](https://www.zhihu.com/equation?tex=%5Cgamma) 要联合起来一起实验才行（也就是说， ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha) 和 ![[公式]](https://www.zhihu.com/equation?tex=%5Cgamma) 的取值会相互影响）。

文章先定义了一个**梯度模长g**：

![[公式]](https://www.zhihu.com/equation?tex=g%3D%7Cp-p%5E%2A%7C%3D%5Cleft%5C%7B+%5Cbegin%7Baligned%7D+1-p+%2C%5Cquad+if%5Cquad++p%5E%2A%3D1%5C%5C++p%2C%5Cquad+if%5Cquad+p%5E%2A%3D0+%5Cend%7Baligned%7D+%5Cright.)

代码如下：

```python3
g = torch.abs(pred.sigmoid().detach() - target)
```

其中 ![[公式]](https://www.zhihu.com/equation?tex=p) 是模型预测的概率，![[公式]](https://www.zhihu.com/equation?tex=p%5E%2A)是 ground-truth的标签， ![[公式]](https://www.zhihu.com/equation?tex=p%5E%2A) 的取值为0或1.

**g正比于检测的难易程度，g越大则检测难度越大。**

至于为什么叫梯度模长，因为g是从交叉熵损失求梯度得来的：

![[公式]](https://www.zhihu.com/equation?tex=L_%7BCE%7D%3D%5Cleft%5C%7B+%5Cbegin%7Baligned%7D+-log%28p%29++%2C%5Cquad+if%5Cquad++p%5E%2A%3D1%5C%5C+-log%281-p%29%2C%5Cquad+if%5Cquad++p%5E%2A%3D0++++%5Cend%7Baligned%7D+%5Cright.)

假定 ![[公式]](https://www.zhihu.com/equation?tex=x) 是样本的输出 ![[公式]](https://www.zhihu.com/equation?tex=+p+%3D+sigmoid%28x%29) ,我们知道 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+p%7D%7B%5Cpartial+x%7D%3Dp%281-p%29) ，

那么 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+L_%7BC+E%7D%7D%7B%5Cpartial+x%7D%3D%5Cfrac%7B%5Cpartial+L_%7BC+E%7D%7D%7B%5Cpartial+p%7D%5Cfrac%7B%5Cpartial+p%7D%7B%5Cpartial+x%7D) ，可以求出

![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+L_%7BCE%7D%7D%7B%5Cpartial+x%7D%3D%5Cleft%5C%7B+%5Cbegin%7Baligned%7D+p-1++%2C%5Cquad+if%5Cquad++p%5E%2A%3D1%5C%5C+p+%2C%5Cquad+if%5Cquad++p%5E%2A%3D0++++%5Cend%7Baligned%7D+%5Cright.) ![[公式]](https://www.zhihu.com/equation?tex=%3Dp-p%5E%2A)

![[公式]](https://www.zhihu.com/equation?tex=g%3D%7C%5Cfrac%7B%5Cpartial+L_%7BCE%7D%7D%7B%5Cpartial+x%7D%7C)

看下图梯度模长与样本数量的关系：

![img](https://pic4.zhimg.com/80/v2-264d18f7c40c4be918b8cecc72f6a02b_720w.jpg)

可以看到，梯度模长接近于0的样本数量最多，随着梯度模长的增长，样本数量迅速减少，但是在梯度模长接近于1时，样本数量也挺多。

GHM的想法是，**我们确实不应该过多关注易分样本，但是特别难分的样本（outliers，离群点）也不该关注啊！**

**这些离群点的梯度模长d要比一般的样本大很多，如果模型被迫去关注这些样本，反而有可能降低模型的准确度！况且，这些样本的数量也很多！**

**那怎么同时衰减易分样本和特别难分的样本呢?太简单了，谁的数量多衰减谁呗！那怎么衰减数量多的呢？简单啊，定义一个变量，让这个变量能衡量出一定梯度范围内的样本数量**——这不就是物理上密度的概念吗？

![[公式]](https://www.zhihu.com/equation?tex=GD%28g%29%3D%5Cfrac%7B1%7D%7Bl_%7B%5Cvarepsilon%7D%28g%29%7D%5Csum_%7Bk%3D1%7D%5E%7BN%7D%7B%5Cdelta_%7B%5Cvarepsilon%7D%28g_%7Bk%7D%2Cg%29%7D)

![[公式]](https://www.zhihu.com/equation?tex=%5Cdelta_+%7B%5Cvarepsilon%7D%28g_%7Bk%7D%2Cg%29) 表明了样本1～N中，梯度模长分布在 ![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%28g-%5Cfrac%7B%5Cvarepsilon%7D%7B2%7D%2C+g%2B%5Cfrac%7B%5Cvarepsilon%7D%7B2%7D%5Cright%29) 范围内的样本个数， ![[公式]](https://www.zhihu.com/equation?tex=l_%7B%5Cepsilon%7D%28g%29) 代表了 ![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%28g-%5Cfrac%7B%5Cvarepsilon%7D%7B2%7D%2C+g%2B%5Cfrac%7B%5Cvarepsilon%7D%7B2%7D%5Cright%29) 区间的长度。

**因此梯度密度** ![[公式]](https://www.zhihu.com/equation?tex=G+D%28g%29) **的物理含义是：单位梯度模长g部分的样本个数。**

接下来就简单了，对于每个样本，把交叉熵CE×该样本梯度密度的倒数即可！

**用于分类的GHM损失** ![[公式]](https://www.zhihu.com/equation?tex=L_%7BG+H+M-C%7D%3D) ![[公式]](https://www.zhihu.com/equation?tex=%3D%5Csum_%7Bi%3D1%7D%5E%7BN%7D+%5Cfrac%7BL_%7BC+E%7D%5Cleft%28p_%7Bi%7D%2C+p_%7Bi%7D%5E%7B%2A%7D%5Cright%29%7D%7BG+D%5Cleft%28g_%7Bi%7D%5Cright%29%7D) ， N是总的样本数量。

## Yolo

yolo的bounding box和ssd的anchor box不一样！！！他不是在图中每一个坐标点生成了一系列的盒子，这个盒子我们把它叫做Anchor，也就是先验盒子！！yolo，没有先验的概念，而是将图片分成了一系列的格子！！ 而这个格子直接预测到的目标的定位的框就是我们所说的bounding box，boungbox的信息直接用途中的格子表示



yolo将一幅图分成s×s个格子（grid cell），如果某个object的中心落在这个格子，那么这个网格就负责这个object。

![img](https://img-blog.csdnimg.cn/20190507093134624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxMjQ5MzU2NTIw,size_16,color_FFFFFF,t_70)

损失函数：

![img](https://img-blog.csdnimg.cn/20190507093134534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxMjQ5MzU2NTIw,size_16,color_FFFFFF,t_70)

yolo v2： 

V2借鉴了faster的思想，引入了anchor，**就像之前说的yolo需要改进的地方，bounding box对一个格子多个目标的预测能力很差**。并且删除了yolo的全连接层，使用anchor预测bounging box 

每个anchor包括25个信息，是否有目标，目标位置，目标种类。 



yolo v3： 

yolov3 对于检测两个距离很近的物体，又或者是距离很近的不同类物体鲁棒性很好，全面超越SSD512。 

v1 v2都不如SSD300 

总的来说 v3比v2性能提高很大，但速度却没有下降，况且对于小目标鲁棒性很强。 



yolo3

**backbone   darknet53**

输入：（batch_size, image_size, image_size, 3）  image_size = 416

输出：（batch_size, 13,13, 255）,（batch_size, 26,26, 255）,（batch_size, 52,52, 255）

下面解释一下255的由来。以13*13的特征图为例。将图片分成13*****13个部分。如果一个物体的中心落在这个格子中，那么这个格子就要负责检测出这个物体。每一个格子会产生3个框对应的值，每个框有85个值，分别是4个坐标，一个得分score,，80个类别概率（因为作者用的coco数据集，总共80类，每个数对应这个类的概率）。坐标表示框的位置，得分是表示这个框是否含有物体。那么对于一个格子，总共需要预测出 ![[公式]](https://www.zhihu.com/equation?tex=3%2A%284%2B1%2B80%29%3D255) 



**框的筛选**

分为三层，13那层对应 ![[公式]](https://www.zhihu.com/equation?tex=13%2A13%2A3%3D507) ，26对应 ![[公式]](https://www.zhihu.com/equation?tex=26%2A26%2A3%3D2028) ，52对应 ![[公式]](https://www.zhihu.com/equation?tex=52%2A52%2A3%3D8112) 总共这么多框。这里作者首先用了一个技巧，首先筛选先验框，把不用的先验框先去掉，减少后续计算量。框的筛选是根据物体得分score以及与ground truth的iou决定的。我们知道，每个物体的中心都会落在确定的一个格子里，对于不存在物体的格子的框都可以忽视掉了。对于存在物体的格子，对于先验框与ground true重合度最高的框，score应该是1（如果不是1，所以就会产生loss），接着我们会计算出predict（真实的框坐标，用于后续loss计算）。



通过不同尺度的特征图来实现不同大小的先验框勾选，    三层表示不同形态的或者说是不同比例的特征框有三个，这个框的形态可以通过对训练集进行聚类的方式来获取。





