---
#layout: post
title: 机器学习评价方法及面试中存在的问题
date: 2021-03-05
Author: 乌卡 
tags: [机器学习, 评价方法, 面试问题]
comments: true
---

这篇主要介绍一些评价方法，同时汇总一些面试中经常遇到的问题

## F1值相关概念介绍

### 都来源于这张表

|       | Positive | Negative |
| ----- | -------- | :------- |
| True  | TP       | TN       |
| False | FP       | FN       |

- Positive/Negative: 正例和负例。这个指的是在二分类问题中的正负样本的意思。
- True/False：模型判定的正例和负例。

#### 准确率（Accuracy）

$acc=\frac {TP+TN}{ALL}$

#### 精确率（Precision）

$precision=\frac{TP}{TP+FP}$

#### 召回率(recall)

$recall=\frac{TP}{TP+TN}$

#### F-Messure

F是Precision和recall的调和值。

$F_1={(\frac{recall^{-1}+precision^{-1}}{2})}^{-1}=2\cdot \frac{precision \cdot recall}{precision+recall}$

其通项公式是：

$F_\beta=(1+\beta^2)\cdot \frac{precision \cdot recall}{(\beta^2 \cdot precision)+recall}$

这个$\beta$就是F1我们常说的1是什么。

如果只是按照TP，TN样本数直接计算呢？

$F_\beta=\frac{(1+\beta^2) \cdot TP}{(1+\beta^2) \cdot TP +\beta^2 \cdot FN +FP}$

#### F1评估方法有啥缺点吗

F-score 独立于其他特征揭示了每个特征的辨别力。 针对第一特征计算一个分数，针对第二特征计算另一个分数。 但它并没有展现两种功能（互信息）组合的信息。 这是 F-score 的主要弱点。

## AUC面积和ROC曲线

### 概念

[详细了解的博客](https://blog.csdn.net/program_developer/article/details/79946787)

ROC全称是“受试者工作特征”(Receiver OperatingCharacteristic)曲线。我们根据学习器的预测结果，把阈值从0变到最大，即刚开始是把每个样本作为正例进行预测，随着阈值的增大，学习器预测正样例数越来越少，直到最后没有一个样本是正样例。在这一过程中，每次计算出两个重要量的值，分别以它们为横、纵坐标作图，就得到了“ROC曲线”。

ROC曲线的纵轴是“真正例率”(True Positive Rate, 简称TPR)，横轴是“假正例率”(False Positive Rate,简称FPR)

这里说下个人的理解，**在大部分的场景下，我们其实就只关心你认为的正确的到底有多少是真正正确的。**所以纵轴才是所谓的真正例，横轴是所谓的假正例。在极端情况下，你全部预测正确，最好的点应该就是(1,0)这个点。

实际情况下是，我们往往不能做到这一点，所以处于正方形的某一个值。我们当然也就能理解为啥曲线要尽可能的靠近(1,0)了。

AUC那就更好理解了。在这里不需要赘述。

