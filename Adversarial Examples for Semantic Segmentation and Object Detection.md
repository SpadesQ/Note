## Adversarial Examples for Semantic Segmentation and Object Detection

[paper](https://arxiv.org/abs/1703.08603 "paper link")

## 论文的目的

之前的研究对抗样本都是针对image classification（单目标），如下图。这篇论文extend adversarial examples
to **semantic segmentation** and **object detection**（多目标）。

![v2-a2d8c6891b2ba56d8014e66db97de0ad_b.jpg](/images/v2-a2d8c6891b2ba56d8014e66db97de0ad_b.jpg)

论文提出Dense Adversary Generation (DAG)网络并在state-of-the-art分割、检测网络测试。  
论文还发现，对抗扰动可以转换用在不同训练数据集的网络、基于不同的体系结构、甚至用于不同的识别任务。 特别是，具有相同架构的网络之间的可转移性比其他情况更为重要。 此外，总结异构扰动往往会带来更好的传递性能，这为黑盒对抗性攻击提供了有效的方法。

## 论文的方法

对于检测：DAG的实现很简单，因为它只涉及为每个目标指定一个对抗标签并执行迭代梯度反向传播。   

对于分割：比检测要复杂，因为目标是更大的数量级，例如，对于具有K个像素的图像，可能的提议的数量是O(K^2)，而像素的数量仅是O（K），其中O（·）是 大O符号。此外，如果仅考虑提议的子集，则在提取新的提议集之后仍然可以正确地识别被扰动的图像（注意，DAG旨在在原始提议上产生识别失败）。 为了增加对抗性攻击的稳健性，我们改变了IOU率以保持优化中增加但仍然合理的提议数量。 在实验中，我们验证当提议在原始图像上足够密集时，很可能在对扰动图像生成的新提议上产生不正确的识别结果。 

迁移性测试：(1) networks with the same architecture but trained with different data;(2) networks with different architectures but trained for the same task; and (3) networks for different tasks. 

### Dense Adversary Generation

 image **X**