## Classification

不考虑检测，不考虑分割，不考虑pysical world，单纯谈谈怎么攻击分类模型。

github上有[cleverhans](https://github.com/tensorflow/cleverhans)和[deepfool](https://github.com/LTS4/DeepFool)这两个库（python的）都可以实现当下主流的对抗样本生成。

下图为论文(Threat of Adversarial Attacks on Deep Learning in Computer Vision: A Survey)总结的12种攻击方法。

<div align=center><img src="/images/Screenshot from 2018-09-18 20-20-34.png"/>lp就是为了让图片肉眼看上去和原图无异</div>

### Box-constrained L-BFGS

Szegedy et al. 2013  
Box-constrained L-BFGS→(R. Fletcher, Practical methods of optimization, John Wiley and Sons,2013.)

<div align=center><img src="/images/1.png"/></div>

这个式子是个正则化式子。 第二项损失函数L(:;:)计算分类损失，让其能被分类成指定的标签label，所以ρ要足够分类错误,但第一项（c > 0）ρ又要使图片看起来无异，所以ρ又必须小。

### Fast Gradient Sign Method (FGSM) 

Goodfellow et al. 2014

<div align=center><img src="/images/2.png"/></div>

where ▽J (:; :; :) 计算损失函数的梯度, sign(:)是sign函数 and <a href="https://www.codecogs.com/eqnedit.php?latex=\epsilon" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\epsilon" title="\epsilon" /></a> 是一个小的标量值，限制了扰动的范数。

使用网络预测的最不可能的目标作为标签l。 然后从原始图像中减去计算出的扰动，使其成为对抗样本。将此方法称为“快速”，因为它不需要迭代过程来计算对抗样本，因此比其他方法快得多。

<div align=center><img src="/images/3.png"/>  

Fast Gradient L2(或者<a href="https://www.codecogs.com/eqnedit.php?latex=L_\infty" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L_\infty" title="L_\infty" /></a>)</div>

_上面的方法都叫做‘one-step’ or ‘one-shot’。_

### Basic & Least-Likely-Class Iterative Methods

相当于上面 “fast”方法的扩展。为什么要迭代？这种迭代量是启发式选择的，对抗样本足以到达<a href="https://www.codecogs.com/eqnedit.php?latex=\epsilon" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\epsilon" title="\epsilon" /></a>max-norm ball边缘而又能掌控计算花费。

<div align=center><img src="/images/6.png"/></div>  

其中α = 1，<a href="https://www.codecogs.com/eqnedit.php?latex=X_0^{adv}=X" target="_blank"><img src="https://latex.codecogs.com/gif.latex?X_0^{adv}=X" title="X_0^{adv}=X" /></a>，迭代次数<a href="https://www.codecogs.com/eqnedit.php?latex=\left&space;\lfloor&space;min(\epsilon&plus;4;&space;1.25\epsilon&space;)&space;\right&space;\rfloor" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\left&space;\lfloor&space;min(\epsilon&plus;4;&space;1.25\epsilon&space;)&space;\right&space;\rfloor" title="\left \lfloor min(\epsilon+4; 1.25\epsilon ) \right \rfloor" /></a>，已经训练好的分类模型θ是固定的。

clip{}对X’每像素裁剪，结果为原图X的<a href="https://www.codecogs.com/eqnedit.php?latex=L_\infty&space;\epsilon-neighbourhood" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L_\infty&space;\epsilon-neighbourhood" title="L_\infty \epsilon-neighbourhood" /></a>。函数如下：

<div align=center><img src="/images/5.png"/></div>  

Madry et al. 指出BIM相当于第一版的投影梯度下降（PGD），一种标准的凸优化方法。

### Jacobian-based Saliency Map Attack (JSMA)

和前面<a href="https://www.codecogs.com/eqnedit.php?latex=l_\infty&space;l_2" target="_blank"><img src="https://latex.codecogs.com/gif.latex?l_\infty&space;l_2" title="l_\infty l_2" /></a>不同于用l0-norm，在物理上，这意味着目标是仅修改图像中的几个像素而不是扰乱整个图像以欺骗分类器。

该算法一次一个地修改原始图像的像素，并监视改变对结果分类的影响。 通过使用网络层的输出的梯度计算saliency map来执行监视。 在该map中，较大的值表示欺骗网络以将l_target预测为图像的标签比原始标签l的可能性更高。 因此，该算法执行有针对性的愚弄。 一旦计算出map，算法就会选择最有效的像素来欺骗网络并改变它。 重复该过程，直到在对抗图像中改变允许像素的最大数量或者愚弄成功为止。

### One Pixel Attack

Su et al.	
顾名思义，只修改图像中的一个像素。通过使用差分进化的概念来计算对抗样本。对于原始图像Ic，创建400向量使其包含任意候选像素的xy-coordinates and RGB，随机修改一个向量，概率预测标签用作评判标准，如此进化，直到找到最好的。差异进化能够生成对抗样本，而无需访问有关网络参数值或梯度的任何信息。 他们的技术需要的唯一输入是目标模型预测的概率标签。

![7.png](/images/7.png)






