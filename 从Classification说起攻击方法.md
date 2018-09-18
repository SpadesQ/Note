## Classification

不考虑检测，不考虑分割，不考虑pysical world，单纯谈谈怎么攻击分类模型。

github上有[cleverhans](https://github.com/tensorflow/cleverhans)和[deepfool](https://github.com/LTS4/DeepFool)这两个库（python的）都可以实现当下主流的对抗样本生成。

<div align=center><img src="/images/Screenshot from 2018-09-18 20-20-34.png"/>lp就是为了让图片肉眼看上去和原图无异</div>

**Box-constrained L-BFGS(R. Fletcher, Practical methods of optimization, John Wiley and Sons,2013.)**  
Szegedy et al. 2013

<div align=center><img src="/images/1.png"/></div>

这个式子是个正则化式子。 第二项损失函数L(:;:)计算分类损失，让其能被分类成指定的标签label，所以ρ要足够分类错误,但第一项（c > 0）ρ又要使图片看起来无异，所以ρ又必须小。

**Fast Gradient Sign Method (FGSM)**  
Goodfellow et al.2014

![2.png](/images/2.png)
