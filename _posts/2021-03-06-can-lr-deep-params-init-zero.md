---
layout: post
title: 机器学习评价方法及面试中存在的问题
date: 2021-03-05
Author: 乌卡 
tags: [机器学习, 评价方法, 面试]
comments: true

---

LR和深度学习的参数可以初始化成0吗？

## LR的参数可以初始化成0吗？

是可以的。

线性回归中间是没有隐藏层的。实际上是直接$y=sigmoid(wx+b)$的映射。我们看下公式

$\begin{aligned}
\mathrm{h}_{w}(\mathrm{x})=& \operatorname{sigmoid}\left(w_{11} x_{1}+w_{21} x_{2}+b\right) \\
&=\frac{1}{1+\mathrm{e}^{\left(w_{11} x_{1}+w_{21} x_{2}+b\right)}}
\end{aligned}$

其损失函数

$\mathrm{J}(w)=-y \log \left(\mathrm{h}_{w}(x)\right)-(1-y) \log \left(1-\mathrm{h}_{w}(x)\right)$

反向传播进行求导之后的一系列计算，是直接可以推出最终的w和b的计算方法的。具体的计算过程如下：

求偏导：

$\begin{array}{c}
\frac{\partial \mathrm{J}(w)}{\partial w}=\frac{\partial \mathrm{J}(w)}{\partial \mathrm{h}_{w}(\mathrm{x})} \cdot \frac{\partial \mathrm{h}_{w}(\mathrm{x})}{\partial w} \\
=\left[-\mathrm{y} \cdot \frac{1}{\mathrm{~h}_{w}(\mathrm{x})}+(1-\mathrm{y}) \cdot \frac{1}{1-\mathrm{h}_{w}(\mathrm{x})}\right] \cdot \frac{\partial \mathrm{h}_{w}(\mathrm{x})}{\partial w}
\end{array}$

其中后项：

$\begin{array}{c}
\frac{\partial \mathrm{h}_{w}(\mathrm{x})}{\partial w}=\left(\frac{1}{1+\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}}}\right)^{\prime}=\frac{\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}} \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime}}{\left(1+\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}}\right)^{2}} \\
=\frac{1}{1+\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}}} \cdot \frac{\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}}}{1+\mathrm{e}^{-\mathrm{w}^{\mathrm{T}} \mathrm{x}}} \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime} \\
=\mathrm{h}_{w}(\mathrm{x}) \cdot\left(1-\mathrm{h}_{w}(\mathrm{x}) \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime}\right.
\end{array}$

带回上式：

$\begin{array}{c}
\frac{\partial \mathrm{J}(w)}{\partial w}=\left[-\mathrm{y} \cdot \frac{1}{\mathrm{~h}_{w}(\mathrm{x})}+(1-\mathrm{y}) \cdot \frac{1}{1-\mathrm{h}_{w}(\mathrm{x})}\right] \cdot \mathrm{h}_{w}(\mathrm{x}) \cdot\left(1-\mathrm{h}_{w}(\mathrm{x}) \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime}\right. \\
=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime}
\end{array}$

对于每个w：

$\begin{aligned}
\frac{\partial \mathrm{J}(w)}{\partial w_{11}} &=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot\left(\mathrm{w}^{\mathrm{T}} \mathrm{x}\right)^{\prime} \\
&=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot \frac{\partial\left(w_{11} x_{1}+w_{21} x_{2}+b\right)}{\partial w_{11}} \\
&=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot x_{1} \\
\frac{\partial \mathrm{J}(w)}{\partial w_{21}} &=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot x_{2} \\
\frac{\partial \mathrm{J}(w)}{\partial b} &=\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right)
\end{aligned}$

最终更新参数可以得到：

$\begin{array}{l}
w_{11}:=w_{11}-\alpha \frac{\partial \mathrm{J}(w)}{\partial w_{11}}=w_{11}-\alpha\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot x_{1} \\
w_{21}:=w_{21}-\alpha \frac{\partial \mathrm{J}(w)}{\partial w_{21}}=w_{21}-\alpha\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right) \cdot x_{2} \\
\quad b:=w_{11}-\alpha \frac{\partial \mathrm{J}(w)}{\partial b}=b-\alpha\left(\mathrm{h}_{w}(\mathrm{x})-\mathrm{y}\right)
\end{array}$

**解释：假设在logistic回归模型中，当我们把W11，W21初始化为0，参数更新中的（∂J/∂W11）和（∂J/∂W21）会因为x1, x2的不同而不同，且不为0，模型的权重可以得到更新。当b的初始化为0的时候，同理，∂J/∂b也不为0，b也可以得到更新。**

## 神经网络参数可以初始化成0吗？

因为有隐藏层了。
我们来考虑一个简化的神经网络模型，它只有两个输入特征，结构为一个输入层，一个隐藏层，一个输出层。。

解释：简单来说，一般的神经网络就是Softmax Regression的组合，而Softmax（多对多，多分类）是多分类版的Logistic Regression（多对一，二分类）。

隐藏层每一个节点$a_1,a_2$都可以看作一个logistic回归的sigmoid激活输出。那么，由$x_1, x_2$获得$a_1$的过程可看作一个上述logistic过程，同理由$a_1, a_2$获得$a_2$的过程也一样，当$w_{11}$和$w_{21}$都初始化为0的时候，依旧是$x_1, x_2$不同所以单看$a_1,a_2$都没问题可以正常更新权值，但$a_1,a_2$节点（sigmoid激活后的输出）相同，这就导致由相同的$a_1, a_2$前向传播到$a_3$以后的反向传播，$w_{13},w_{23}$不更新。

## 参考文章

https://blog.csdn.net/weixin_43627748/article/details/113994231