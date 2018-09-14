> 过去笔记以翻译为主，现在重新定义笔记，笔记尽可能精简，重要的信息为了不曲解愿意将保留原文

## 论文的目的

研究 adversarial examples in the physical world，选了路标"stop"贴黑白纸条（模拟小广告=。=)

<div align=center><img src="/images/22333.png"/></div>

不可能随便贴去试哪个可以攻击，所以理论是在digital world研究。

然而physical world是复杂的：distance and angle拍摄的距离和角度、Spatial Constraints扰动的空间限制（只能在路牌上贴小纸条，不可能贴外面去）、physical limits on how imperceptible perturbations（确保相机拍摄能感知到扰动）、Fabrication Error（digital world得出的扰动要打印出来贴到physical world，所以要确保打印无误）。

所以在digital world研究要考虑以上四个因素。

## 论文的方法

定义perturbation δ, perturbed instance x‘ = x+δ, target classifier fθ (·), y∗ is the target class, H(x + δ, x)表示对抗样本和真实样本距离函数（尽可能小）。

<div align=center><img src="/images/22444.png"/></div>

Here J(·,·) is the loss function, which measures the difference between the model’s prediction and the target label y∗ . λ is a hyper-parameter that controls the regularization of the distortion. We specify the distance function H as ||δ||p ,denoting the lp norm of δ.

这个公式的意思是一方面δ要小，另一方面扰动δ还要确保能分类错误。

好，接下来考虑那四个挑战。

We model the distribution of images containing object o under **both physical and digital transformations**X, 意思是包含实际的物理条件变化（包括changing distances, angles, and lightning）以及数字变换（包括change the brightness, and add spatial transformations）, 从这个分布采样xi。 过去的研究都只有数字变换对于本论文是不够的。

To ensure that the perturbations are only applied to the surface area of the target object o(considering the spatial constraints and physical limits on imperceptibility), we introduce a **mask**.（别把小广告贴外面了，另外人还得能看见小广告，但是还不能让人认不出"stop"）

所以**shape the mask to look like graffiti**（mask看起来像涂鸦）。Formally, the perturbation mask is a matrix Mx whose dimensions are the same as the size of input to the road sign classifier.(mask是矩阵Mx，其尺寸与道路标志分类器的输入尺寸相同。)Mx contains zeroes in regions where no perturbation is added, and ones in regions where the perturbation is added during optimization.**看下图右侧理解**。

<div align=center><img src="/images/Screenshot from 2018-09-13 20-33-59.png"/></div>

Specifically, we use the following pipeline to **discover mask positions**:(小广告应该贴哪=。=)      
(1) Compute perturbations using the L 1 regularization and with a mask that occupies the entire surface area of the sign. L 1 makes the optimizer favor a sparse perturbation vector, therefore concentrating the perturbations on regions that are most vulnerable. Visualizing the resulting perturbation provides guidance on mask placement.   
使用L1正则化和占据整个表面区域的mask来计算扰动。 L 1使得优化器倾向于稀疏扰动向量，因此将扰动集中在最易受攻击的区域上。 可视化产生的扰动提供了掩模放置的指导。     
(2) Recompute perturbations using L 2 with a mask positioned on the vulnerable regions identified from the earlier step.  
使用L2重新计算扰动，mask位于从前一步骤识别的易受攻击区域上。

好了考虑打印的影响。To account for fabrication error, add an additional term to our objective function that models printer color reproduction errors.**This term is based upon the Non-Printability Score (NPS) by Sharif et al.**. Given a set of printable colors (RGB triples) P and a set R(δ) of (unique) RGB
triples used in the perturbation that need to be printed out in physical world, the non-printability score is given by:

<div align=center><img src="/images/22555.png"/></div>

最终优化式子：

<div align=center><img src="/images/22666.png"/></div>

**Ti (·)** to denote the alignment function that maps transformations on the object to transformations on the perturbation (e.g.**if the object is rotated, the perturbation is rotated as well**).

Finally, an attacker will print out the optimization result on paper, cut out the perturbation (Mx), and put it onto the target object o.（小广告裁剪下来贴到路牌上=。=）

## 论文的实验

数据集：LISA, a U.S. traffic sign dataset containing 47 different road signs，论文用17种训练

分类器：LISA-CNN(consists of three convolutional layers and an FC layer 3卷积层1全连接层很简单的分类器)and GTSRB-CNN

实验分两部分Stationary (Lab) Tests和Drive-By (Field) Tests. With a perturbation in the form of only **black and white stickers**,we attack a real **stop** sign, causing targeted misclassification in **100% of the images obtained in lab settings**, and in **84.8% of the captured video frames obtained on a moving vehicle (field test)** for the target classifier.

Stationary (Lab) Tests很简单，直接贴小广告，看攻击分类器的成功率。

<div align=center><img src="/images/22777.png"/></div>

公式表示原图分类正确的情况下添加扰动后分类错误的成功率。下图是和包括**用论文的算法**重建Kurakin et al.工作（修改海报，把整个海报贴上去）比较的结果（Object-Constrained Poster-Printing Attacks）

![22999.png](/images/22999.png)

Drive-By Testing.计算攻击成功率公式不变, 车速between 0 mph and20 mph, 每10帧计算一次。下图是结果同样和**用论文的算法**重建Kurakin et al.比较。

![22000.png](/images/22000.png)

To show the generality of our approach, we generate the robust physical adversarial example by manipulating general physical objects, such as a microwave. We show that the **pre-trained Inception-v3**classifier misclassifies the microwave as “phone" by adding a single sticker.

## Q & A

