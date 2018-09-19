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

<div align=center><img src="/images/4.png"/></div>  

其中<a href="https://www.codecogs.com/eqnedit.php?latex=I^0_\rho=&space;I_c" target="_blank"><img src="https://latex.codecogs.com/gif.latex?I^0_\rho=&space;I_c" title="I^0_\rho= I_c" /></a>，迭代次数<a href="https://www.codecogs.com/eqnedit.php?latex=\left&space;\lfloor&space;min(\epsilon&plus;4;&space;1.25\epsilon&space;)&space;\right&space;\rfloor" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\left&space;\lfloor&space;min(\epsilon&plus;4;&space;1.25\epsilon&space;)&space;\right&space;\rfloor" title="\left \lfloor min(\epsilon+4; 1.25\epsilon ) \right \rfloor" /></a>，已经训练好的分类模型θ是固定的。

clip{}对X’每像素裁剪，结果为原图X的<a href="https://www.codecogs.com/eqnedit.php?latex=L_\infty&space;\epsilon-neighbourhood" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L_\infty&space;\epsilon-neighbourhood" title="L_\infty \epsilon-neighbourhood" /></a>。函数如下：

<div align=center><img src="/images/5.png"/></div>  

Madry et al. 指出BIM相当于第一版的投影梯度下降（PGD），一种标准的凸优化方法。
