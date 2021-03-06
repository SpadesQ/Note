# YOLOv1到v3的进化之路

相对于传统的分类问题，目标检测显然更符合现实需求，因为往往现实中不可能在某一个场景只有一个物体，因此目标检测的需求变得更为复杂，不仅仅要求算法能够检验出是什么物体，还需要确定这个物体在图片哪里。

在这一过程中，目标检测经历了一个高度的符合人类的直觉的过程。既需要识别出目标的位置，将图片划分成小图片扔进算法中去，当算法认为某物体在这个小区域上之时，那么检测完成。那我们就认为这个物体在这个小图片上了。而这个思路，正是比较早期的目标检测思路，比如R-CNN。

后来的Fast R-CNN，Faster R-CNN[16]虽有改进，比如不再是将图片一块块的传进CNN提取特征，而是整体放进CNN提取特征图后，再做进一步处理，但依旧是整体流程分为 ‘区域提取’和‘目标分类’两部分（two-stage），这样做的一个特点是虽然确保了精度，但速度非常慢，于是以YOLO（You only look once）为主要代表的这种一步到位(one-stage)即端到端的目标检测算法应运而生了。

[基于深度学习的目标检测技术演进：R-CNN、Fast R-CNN、Faster R-CNN ](https://www.cnblogs.com/skyfsm/p/6806246.html)

RCNN  
　　1.在图像中确定约1000-2000个候选框 (使用选择性搜索)   
　　2.每个候选框内图像块缩放至相同大小，并输入到CNN内进行特征提取        
　　3.对候选框中提取出的特征，使用分类器判别是否属于一个特定类  
　　4.对于属于某一特征的候选框，用回归器进一步调整其位置

Fast RCNN   
　　1. 在图像中确定约1000-2000个候选框 (使用选择性搜索)  
　　2. 对整张图片输进CNN，得到feature map  
　　3. 找到每个候选框在feature map上的映射patch，将此patch作为每个候选框的卷积特征输入到SPP layer和之后的层  
　　4. 对候选框中提取出的特征，使用分类器判别是否属于一个特定类  
　　5. 对于属于某一特征的候选框，用回归器进一步调整其位置  

Faster RCNN  
　　1. 对整张图片输进CNN，得到feature map  
　　2. 卷积特征输入到RPN，得到候选框的特征信息  
　　3. 对候选框中提取出的特征，使用分类器判别是否属于一个特定类  
　　4. 对于属于某一特征的候选框，用回归器进一步调整其位置  
  
## YOLO V1 一步检测的开山之作

### 1. YOLO v1的核心思想

YOLO 的核心思想就是利用整张图作为网络的输入，直接在输出层回归 bounding box（边界框） 的位置及其所属的类别。

### 2. YOLO v1的实现方法

<div align=center><img src="/images/20180606164153997.png"/></div>

- 将一幅图像分成 SxS 个网格（grid cell），如果某个 object 的中心落在这个网格中，则这个网格就负责预测这个 object。
- 每个网格要预测 B 个 bounding box，每个 bounding box 除了要回归自身的位置之外，还要附带预测一个 confidence 值。

这个 confidence 代表了所预测的 box 中含有 object 的置信度和这个 box 预测的有多准这两重信息，其值是这样计算的： 

<div align=center><img src="/images/20180606164218784.png"/></div>

这个置信度并不只是该边界框是待检测目标的概率，而是该边界框是**待检测目标的概率乘上该边界框和真实位置的[IOU](https://blog.csdn.net/hysteric314/article/details/54093734)**（框之间的交集除以并集）的积。通过乘上这个交并比，反映出该边界框预测位置的精度。

每个边界框对应于5个输出，分别是x，y，w，h和置信度。其中x，y代表边界框的中心离开其所在网格单元格边界的偏移。w，h代表边界框真实宽高相对于整幅图像的比例。**x，y，w，h这几个参数都已经被限制到了区间[0,1]上。**除此以外，**每个单元格还产生C( categories)个条件概率**。

在test的**非极大值抑制**阶段，每个网格预测的 class 信息和 bounding box 预测的 confidence信息相乘，就得到每个 bounding box 的 class-specific confidence score，即下式衡量该框是否应该予以保留。

<div align=center><img src="/images/20180606164339450.png"/></div>

这里再介绍一下非极大值抑制算法（non maximum suppression, NMS），这个算法不单单是针对Yolo算法的，而是所有的检测算法中都会用到。NMS算法主要解决的是一个目标被多次检测的问题，如下图中人脸检测，可以看到人脸被多次检测，但是其实我们希望最后仅仅输出其中一个最好的预测框，比如对于美女，只想要红色那个检测结果。那么可以采用NMS算法来实现这样的效果：首先从所有的检测框中找到置信度最大的那个框，然后挨个计算其与剩余框的IOU，如果其值大于一定阈值（重合度过高），那么就将该框剔除；然后对剩余的检测框重复上述过程，直到处理完所有的检测框。Yolo预测过程也需要用到NMS算法。

<div align=center><img src="/images/Screenshot from 2018-09-10 15-57-58.png"/></div>

举例说明: 在 PASCAL VOC 中，图像输入为 448x448，取 S=7，B=2，一共有20 个类别（C=20），则输出就是 S x S x (5*B+C) = 7x7x30 的一个 tensor。

<div align=center><img src="/images/Screenshot from 2018-09-10 16-10-22.png"/></div>

每一个栅格还要预测C个 conditional class probability（条件类别概率）：Pr(Classi|Object)。即在一个栅格包含一个Object的前提下，它属于某个类的概率。我们只为每个栅格预测一组（C个）类概率，而不考虑框B的数量。**注意：conditional class probability信息是针对每个网格的。confidence信息是针对每个bounding box的。**

<div align=center><img src="/images/20170420214004907.jpeg"/></div>

每个栅格的conditional class probabilities与每个 bounding box的 confidence相乘:

<div align=center><img src="/images/Screenshot from 2018-09-10 16-16-19.png"/></div>

<div align=center><img src="/images/Screenshot from 2018-09-10 16-28-24.png"/></div>

非极大值抑制：
<div align=center><img src="/images/Screenshot from 2018-09-10 16-30-51.png"/></div>
<div align=center><img src="/images/Screenshot from 2018-09-10 16-31-09.png"/></div>

输出:
<div align=center><img src="/images/Screenshot from 2018-09-10 16-33-25.png"/></div>

**网络结构：**

<div align=center><img src="/images/20180606164310266.png"/></div>

**Bounding Box Normalization:**

YOLO在实现中有一个重要细节，即对bounding box的坐标(x, y, w, h)进行了normalization，以便进行回归。作者认为这是一个非常重要的细节。在原文2.2 Traing节中有如下一段：

> Our final layer predicts both class probabilities and bounding box coordinates.We normalize the bounding box width and height by the image width and height so that they fall between 0 and 1.We parametrize the bounding box x and y coordinates to be offsets of a particular grid cell location so they are also bounded between 0 and 1.

<div align=center><img src="/images/20170603134214525.jpeg"/>
  <p>SxS网格与bounding box关系（图中S=7，row=4且col=1）</p></div>
  
在YOLO中输入图像被分为SxS网格。假设有一个bounding box(如上图红框），其中心刚好落在了(row,col)网格中，则这个网格需要负责预测整个红框中的dog目标。假设图像的宽为width_image，高为height_image；红框中心在(xc，yc)，宽为width_box，高为height_box那么：

(1) 对于bounding box的宽和高做如下normalization，使得输出宽高介于0~1：

<div align=center><img src="/images/20170605002831961.jpeg"/></div>

(2) 使用(row, col)网格的offset归一化bounding box的中心坐标：

<div align=center><img src="/images/20170603134345672.jpeg"/></div>
  
经过上述公式得到的normalization的(x, y, w, h)，再加之前提到的confidence，共同组成了一个真正在网络中用于回归的bounding box；而当网络在Test阶段(x, y, w, h)经过反向解码又可得到目标在图像坐标系的框，解码代码在darknet detection_layer.c中的get_detection_boxes()函数，关键部分如下：
```
    boxes[index].x = (predictions[box_index + 0] + col) / l.side * w;    
    boxes[index].y = (predictions[box_index + 1] + row) / l.side * h;    
    boxes[index].w = pow(predictions[box_index + 2], (l.sqrt?2:1)) * w;    
    boxes[index].h = pow(predictions[box_index + 3], (l.sqrt?2:1)) * h;    
```
而w和h就是图像宽高，l.side是上文中提到的S。 

### 3. YOLO v1的损失函数

YOLO v1全部使用了均方差（mean squared error）作为损失（loss）函数。由三部分组成：坐标误差、IOU误差和分类误差。

<div align=center><img src="/images/20180606164516310.png"/></div>

- 更重视8维的坐标预测，给这些损失前面赋予更大的 loss weight, 记为在 pascal VOC 训练中取 5。
- 对没有 object 的 box 的 confidence loss，赋予小的 loss weight，记为在 pascal VOC 训练中取 0.5。
- 有 object 的 box 的 confidence loss 和类别的 loss 的 loss weight 正常取 1。
- 对不同大小的 box 预测中，相比于大 box 预测偏一点，小 box 预测偏一点肯定更不能被忍受的。而 sum-square error loss 中对同样的偏移 loss 是一样。为了缓和这个问题，作者用了一个比较取巧的办法，就是将 box 的 width 和 height 取平方根代替原本的 height 和 width。这个参考下面的图很容易理解，小box的横轴值较小，发生偏移时，反应到y轴上相比大 box 要大。（也是个近似逼近方式）

<div align=center><img src="/images/20180606164449500.png"/></div>

### 4. YOLO v1的缺点
 
- 由于输出层为全连接层，因此在检测时，YOLO 训练模型只支持与训练图像相同的输入分辨率。    
- 虽然每个格子可以预测 B 个 bounding box，但是最终只选择只选择 IOU 最高的 bounding box 作为物体检测输出，即每个格子最多只预测出一个物体。当物体占画面比例较小，如图像中包含畜群或鸟群时，每个格子包含多个物体，但却只能检测出其中一个。这是 YOLO 方法的一个缺陷。
- YOLO 的损失函数中，大物体 IOU 误差和小物体 IOU 误差对网络训练中 loss 贡献值接近（虽然采用求平方根方式，但没有根本解决问题）。因此，对于小物体，小的 IOU 误差也会对网络优化过程造成很大的影响，从而降低了物体检测的定位准确性。


## YOLOv2/YOLO9000 更准、更快、更强

YOLO v1对于bounding box的定位不是很好，在精度上比同类网络还有一定的差距。作者希望改进的方向是改善 [recall，提升定位的准确度，同时保持分类的准确度](https://blog.csdn.net/hysteric314/article/details/54093734)。YOLO V2在V1基础上做出改进：

- 受到Faster RCNN方法的启发，引入了anchor。使用了K-Means方法，对anchor数量进行了讨论。
- 修改了网络结构，去掉了全连接层，改成了全卷积结构。
- 训练时引入了世界树（WordTree）结构，将检测和分类问题做成了一个统一的框架，并且提出了一种层次性联合训练方法，将ImageNet分类数据集和COCO检测数据集同时对模型训练。

### 更准

**Batch Normalization**

使用 Batch Normalization 对网络进行优化，让网络提高了收敛性，同时还消除了对其他形式的正则化（regularization）的依赖。通过对 YOLO 的每一个卷积层增加 Batch Normalization，最终使得 mAP 提高了 2%，同时还使模型正则化。使用 Batch Normalization 可以从模型中去掉 Dropout，而不会产生过拟合。

**High resolution classifier**

目前业界标准的检测方法，都要先把分类器（classiﬁer）放在ImageNet上进行预训练。从 Alexnet 开始，大多数的分类器都运行在小于 256*256 的图片上。而现在 YOLO 从 224*224 增加到了 448*448，这就意味着网络需要适应新的输入分辨率。

为了适应新的分辨率，YOLO v2 的分类网络以 448*448 的分辨率先在 ImageNet上进行微调，微调 10 个 epochs，让网络有时间调整滤波器（filters），好让其能更好的运行在新分辨率上，还需要调优用于检测的 Resulting Network。最终通过使用高分辨率，mAP 提升了 4%。

**Convolution with anchor boxes**

YOLO(v1)使用全连接层数据进行bounding box预测（将全连接层转换为S*S*(B*5+20)维的特征），这会丢失较多的空间信息，导致定位不准。 

YOLOv2借鉴了Faster R-CNN中的anchor思想： 简单理解为卷积特征图上进行滑窗采样，每个中心预测9种不同大小和比例的建议框。由于都是卷积不需要reshape，很好的保留了空间信息，最终特征图的每个特征点和原图的每个cell一一对应。而且用预测相对偏移（offset）取代直接预测坐标简化了问题，方便网络学习。总的来说就是移除全连接层（以获得更多空间信息）使用 anchor boxes去预测 bounding boxes。并且，YOLOv2将预测类别的机制从空间位置(cell)中解耦，由anchor box同时预测类别和坐标(YOLO是由每个cell来负责预测类别，每个cell对应的2个bounding负责预测坐标)。

![gqYMaEH.png](/images/gqYMaEH.png)

关于Faster R-CNN中的anchor：
![20180214224221964.png](/images/20180214224221964.png)

其实featuremap对于anchorbox的生成的贡献就是提供了一个中心点而已，featuremap每个位置上的点，就对应一个anchorbox的中心，然后呢，知道了这么多中心点，根据base size，scales，aspect ratios就可以算出来一个矩形的长和宽。矩形的中心点就是featuremap上的那个点对应原图上的点。

这个矩形的长和宽的计算很好理解，但是怎么得到featuremap的点对应原图（这里原图指的是resize之后的图，后面也都这么说，因为是resize之后的图参与计算，得到的location信息是图上的相对比例的坐标，所以不用真正的原图也没关系）上是哪个点呢？原图过了网络之后，大概缩放比例就是(?)倍,有一个stride参数，也就是把featuremap的坐标平移一下（乘?）就得到相对于原图的坐标了。

**Dimension clusters**

Anchor boxes的宽高维度往往是精选的先验框（hand-picked priors）也就是说人工选定的先验框。虽然在训练过程中网络也会学习调整框的宽高维度，最终得到准确的bounding boxes。但是，如果一开始就选择了更好的、更有代表性的先验框维度，那么网络就更容易学到准确的预测位置。为了优化，在训练集的 Bounding Boxes 上跑一下 k-means聚类，来找到一个比较好的值。

<div align=center><img src="/images/20180527230952967756.png"/></div>

可以看出k=5在模型复杂度与召回率之间取一个折中值。

**Direct location prediction**

用 Anchor Box 的方法，会让 model 变得不稳定，尤其是在最开始的几次迭代的时候。大多数不稳定因素产生自预测 Box 的（x,y）位置的时候。按照之前 YOLO的方法，网络不会预测偏移量，而是根据 YOLO 中的网格单元的位置来预测坐标，这就让 Ground Truth 的值介于 0 到 1 之间。在区域建议网络中，预测 (x,y) 以及 tx，ty 使用的是如下公式：
<div align=center><img src="/images/20161229113738852.png"/></div>
作者应该是把加号写成了减号。理由如下，anchor的预测公式来自于Faster-RCNN：
<div align=center><img src="/images/20170417171727693.png"/></div>

公式中，符号的含义解释一下：x 是坐标预测值，xa 是anchor坐标（预设固定值），x∗ 是坐标真实值（标注信息），其他变量 y，w，h 以此类推，t 变量是偏移量。变形后是加号。

上上面公式的理解为：当预测 tx=1，就会把box向右边移动一定距离（具体为anchor box的宽度），预测 tx=−1，就会把box向左边移动相同的距离。因此每个位置预测的边界框可以落在图片任何位置，这导致模型的不稳定性，在训练时需要很长时间来预测出正确的offsets。

所以，YOLOv2弃用了这种预测方式，而是沿用YOLOv1的方法，就是预测边界框中心点相对于对应cell左上角位置的相对偏移值，为了将边界框中心点约束在当前cell中，使用sigmoid函数处理偏移值，这样预测的偏移值在(0,1)范围内（每个cell的尺度看做1）。

网络在每一个网格单元中预测出 5 个 Bounding Boxes，每个 Bounding Boxes 有五个坐标值 tx，ty，tw，th，t0(txty变量是相对于grid cell的偏移量,twth是相对于anchor的偏移量)，他们的关系见下图（Figure3）。假设一个网格单元对于图片左上角的偏移量是 cx、cy，Bounding Boxes Prior 的宽度和高度是 pw、ph，那么预测的结果见下图右面的公式： 

<div align=center><img src="/images/20180606164911315.png"/></div>

这几个公式参考上面Faster-RCNN和YOLOv1的公式以及下图就比较容易理解。tx,ty 经sigmod函数处理过，取值限定在了0~1，实际意义就是使anchor只负责周围的box，有利于提升效率和网络收敛。σ 函数的意义没有给，但估计是把归一化值转化为图中真实值，使用 e 的幂函数是因为前面做了**ln**计算，因此，σ(tx)是bounding box的中心相对栅格左上角的横坐标，σ(ty)是纵坐标，σ(to)是bounding box的confidence score。

**Fine-Grained Features**

 YOLOv1在对于大目标检测有很好的效果，但是对小目标检测上，效果欠佳。为了改善这一问题，作者参考了Faster R-CNN和SSD的想法，在不同层次的特征图上获取不同分辨率的特征。作者将上层的(前面26×26)高分辨率的特征图（feature map）直接连到13×13的feature map上。把26×26×512转换为13×13×2048，并拼接住在一起使整体性能提升1%。
 
**Multi-Scale Training**

和GoogleNet训练时一样，为了提高模型的鲁棒性（robust），在训练的时候使用多尺度[6]的输入进行训练。YOLOv2 每迭代几次都会改变网络参数。每 10 个 Batch，网络会随机地选择一个新的图片尺寸，由于使用了下采样参数是  32，所以不同的尺寸大小也选择为 32 的倍数 {320，352…..608}，最小 320*320，最大 608*608，网络会自动改变尺寸，并继续训练的过程。

### 更快

大多数目标检测的框架是建立在VGG-16上的，VGG-16在ImageNet上能达到90%的top-5（最后概率向量最大的前五名中出现了正确概率即为预测正确），但是单张图片需要30.69 billion 浮点运算，YOLO2是依赖于DarkNet-19的结构，该模型在ImageNet上能达到91%的top-5，并且单张图片只需要5.58 billion 浮点运算，大大的加快了运算速度。

YOLOv2去掉YOLOv1的全连接层，同时去掉YOLO v1的最后一个池化层，增加特征的分辨率，修改网络的输入，保证特征图有一个中心点，这样可提高效率。并且是以每个anchor box来预测物体种类的。

作者将分类和检测分开训练，在训练分类时，以Darknet-19为模型在ImageNet上用随机梯度下降法（Stochastic gradient descent）跑了160epochs，跑完了160 epochs后，把输入尺寸从224×224上调为448×448，这时候学习率调到0.001，再跑了10 epochs， DarkNet达到了top-1准确率76.5%，top-5准确率93.3%。

在训练检测时，作者把分类网络改成检测网络，去掉原先网络的最后一个卷积层，取而代之的是使用3个3×3x1024的卷积层，并且每个新增的卷积层后面接1×1的卷积层，数量是我们要检测的类的数量。

### 更强

论文提出了一种联合训练的机制：使用识别数据集训练模型识别相关部分，使用分类数据集训练模型分类相关部分。

众多周知，检测数据集的标注要比分类数据集打标签繁琐的多，所以ImageNet分类数据集比VOC等检测数据集高出几个数量级。所以在YOLOv1中，边界框的预测其实并不依赖于物体的标签，YOLOv2实现了在分类和检测数据集上的联合训练。对于检测数据集，可以用来学习预测物体的边界框、置信度以及为物体分类，而对于分类数据集可以仅用来学习分类，但是其可以大大扩充模型所能检测的物体种类。

作者选择在COCO和ImageNet数据集上进行联合训练，遇到的第一问题是两者的类别并不是完全互斥的，比如"Norfolk terrier"明显属于"dog"，所以作者提出了一种层级分类方法（Hierarchical classification），根据各个类别之间的从属关系（根据WordNet）建立一种树结构WordTree，结合COCO和ImageNet建立的词树（WordTree）如下图所示：

<div align=center><img src="/images/20180527230953639656.png"/></div>

WordTree中的根节点为"physical object"，每个节点的子节点都属于同一子类，可以对它们进行softmax处理。在给出某个类别的预测概率时，需要找到其所在的位置，遍历这个路径，然后计算路径上各个节点的概率之积。

在训练时，如果是检测样本，按照YOLOv2的loss计算误差，而对于分类样本，只计算分类误差。在预测时，YOLOv2给出的置信度就是  ，同时会给出边界框位置以及一个树状概率图。在这个概率图中找到概率最高的路径，当达到某一个阈值时停止，就用当前节点表示预测的类别。

## YOLO v3 集大成之作

改进之处：

- 多尺度预测 （类FPN）
- 更好的基础分类网络（类ResNet）和分类器 darknet-53
- 分类器-类别预测：

### 多尺度预测

原来的YOLO v2有一个层叫：passthrough layer，假设最后提取的feature map的size是13*13，那么这个层的作用就是将前面一层的26*26的feature map和本层的13*13的feature map进行连接，有点像ResNet。这样的操作也是为了加强YOLO算法对小目标检测的精确度。这个思想在YOLO v3中得到了进一步加强，在YOLO v3中采用类似FPN的上采样（upsample）和融合做法（最后融合了3个scale，其他两个scale的大小分别是26*26和52*52），在多个scale的feature map上做检测，对于小目标的检测效果提升还是比较明显的。

有别于yolov2，这里作者将每个grid cell预测的边框数从yolov2的5个减为yolov3的3个。最终输出的tensor维度为N × N × [3 ∗ (4 + 1 + 80)] 。其中N为feature map的长宽，3表示3个预测的边框，4表示边框的tx,ty,tw,th，1表示预测的边框的置信度，80表示分类的类别数。

和yolov2一样，anchor的大小作者还是使用kmeans聚类得出。在coco数据集上的9个anchor大小分别为：(10× 13); (16× 30); (33× 23); (30× 61); (62× 45); (59×119); (116 × 90); (156 × 198); (373 × 326) 

### darknet-53

和yolov2的19层的骨架（Darknet-19 ）不同，yolov3中，作者提出了53层的骨架（Darknet-53 ），并且借鉴了ResNet的shortcut结构。

<div align=center><img src="/images/20180527230953899432.png"/></div>

### 分类损失函数

yolov3中将yolov2中多分类损失函数softmax cross-entropy loss 换为2分类损失函数binary cross-entropy loss 。因为当图片中存在物体相互遮挡的情形时，一个box可能属于好几个物体，而不是单单的属于这个不属于那个，这时使用2分类的损失函数就更有优势。

## References

http://www.mamicode.com/info-detail-2314392.html  
http://dwz.cn/7ZGrif
https://blog.csdn.net/baobei0112/article/details/78285141
