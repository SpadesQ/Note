> 过去笔记以翻译为主，现在重新定义笔记，笔记尽可能精简，重要的信息将保留原文

## 论文的目的

研究 adversarial examples in the physical world，选了路标"stop"贴黑白纸条（模拟小广告=。=）

![22333.png](/images/22333.png)

不可能随便贴去试哪个可以攻击，所以理论是在digital world研究。

然而physical world是复杂的：distance and angle拍摄的距离和角度、Spatial Constraints扰动的空间限制（只能在路牌上贴小纸条，不可能贴外面去）、physical limits on how imperceptible perturbations（确保相机拍摄能感知到扰动）、Fabrication Error（digital world得出的扰动要打印出来贴到physical world，所以要确保打印无误）。

所以在digital world研究要考虑以上四个因素。

## 论文的方法

定义perturbation δ, perturbed instance x‘ = x+δ, target classifier fθ (·), y∗ is the target class

![22444.png](/images/22444.png)

Here J(·,·) is the loss function, which measures the difference between the model’s prediction and the target labely∗ . λ is a hyper-parameter that controls the regularization of the distortion. We specify the distance function H as ||δ||p ,denoting the lp norm of δ.

> With a perturbation in the form of only **black and white stickers**,we attack a real **stop** sign, causing targeted misclassification in **100% of the images obtained in lab settings**, and in **84.8% of the captured video frames obtained on a moving vehicle (field test)** for the target classifier.



## Q & A

1.怎么保证 δ 就是 黑白方块

<div align=center><img src="/images/Screenshot from 2018-09-13 20-33-59.png"/></div>

2.怎么保证打印下来贴到物理标志上效果一样

<div align=center><img src="/images/Screenshot from 2018-09-13 20-43-45.png"/></div>

