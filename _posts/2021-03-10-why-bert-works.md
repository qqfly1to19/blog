---
layout: post
title: 为什么Berts这么牛逼
date: 2021-03-10
Author: 乌卡 
tags: [PTM, Bert, 面试, 预训练模型]
comments: true
---

Berts作为承前启后的预训练模型，直接将NLP代入了一个新的阶段。而通过pre-train -> fine-tune的新的任务方式，也成为了NLP任务中新的范式。这篇文章主要是记录关于Bert面试中经常提到的一些问题。

## Bert基本原理是什么

Bert出处：Pre-training of Deep Bidirectional Transformers for Language Understanding

整体是一个自编码语言模型(Auto-encoder LM)，他设计了两个预训练任务。

- MLM。基于掩码的语言模型。在Bert中是Mask一个词。根据这个任务有很多变种，有连续mask的，也有预测mask中的位置的。以后我们介绍下。
- NSP：Next Sentence Prediction。这个有很多变种，albert里面用的是SOP：Sentence Order Prediction。除此之外，我们也能自己设计一些新的Prediction任务。

之后他在不同的任务上进行fine-tune。

- 句子对任务
- 句子分类任务

![image-20210310122131007](https://tva1.sinaimg.cn/large/008eGmZEly1goeoxxuyatj30y50eegpg.jpg)

### Bert在输入端相比于原来的transformer有啥变化吗？

多了一个Segment Embedding。Segment Embedding是句子向量的意思。为什么会多一个Segment Embedding呢？因为Bert的预训练和Pretrain任务中，是有一些句子对任务，句子分类任务的。这个句子对任务是有上一句和下一句之分的，他考虑了句子之间的书讯冠旭，通过[SEP]标识进行标记。所以就会有句子A和句子B这样的字眼。

### 为什么这些Embedding可以相加输入呢

实际上，我们在很多场景下，信息都是叠加态的。在语音处理的领域更为常见，这说明了Embedding叠加输入仍可以表示一个正常的信号。同时，在其他的一些场景，例如静态词向量，一个词的向量其实可以认为是多个上下文的一个叠加态平均值。

你也可以使用其他的方式，例如concatenate，但是这样会造成维度加大，计算更复杂。

![image-20210310122034011](https://tva1.sinaimg.cn/large/008eGmZEly1goeowzgsjbj30ry08xgmx.jpg)



## Bert中使用transformer相比于之前的ELMOS好在哪

Bert使用Transformer，而不是用LSTM(elmos)，RNN作为特征抽取器。关于Recurrent抽取器和Transformer的比较在此不做对比。我们知道的是，Recurrent抽取器的问题就是长时依赖不好做，训练师状态也是依赖的，所以速度比较慢。总体而言导致了抽取效果不是很好。同时，Recurrent抽取器不便于并行，也让他很难去叠加到深层。这个使用multi-head self-attention可以解决这些问题。

## Bert 为什么使用Word piece，有什么好处

wordpiece实际上就是把一个单词给切分开，这个在中文中是不存在的，但是在后续的改进中，中文在处理分词任务时也使用了这种方法。

在word piece中，一个词：lisening会被切分成listen ##ing。相当于把前缀、后缀(词根)给取出来单独作为一个词。这是因为在英语中，词的后缀变化比较多，往往代表的是一个词的词性变化。这样做的好处就是能够缩小词表，减少OOV出现的情况。

同时，通过词的切分，语义上的损失是最小的。

## 什么是BERT的半精度训练

[参考文献1](http://market.itcgb.com/Contents/Intel/OR_AI_BJ/images/Brian_DeepLearning_LowNumericalPrecision.pdf)

[参考文献2](https://fyubang.com/2019/08/26/fp16/)

这是一种模型压缩的方法，可以加快模型的训练和推理过程，同时可以显著降低显存的占用率。其实就是FP32 -> FP16。$10^{-38}$ -> $6^{-5}$

带来的问题，量化误差：

- 狭窄的表示范围带来的溢出错误（上溢和下溢）
- 精度不足带来的四舍五入。

解决方案：

- 混合精度训练。内存中用FP16做存储和乘法加速计算，用FP32做累加避免舍入误差
- 放大损失：激活梯度的值太小，造成下溢。
  - 反向传播时，损失手动增大$s^k$倍。
  - 反向传播后，权重梯度缩小$2^k$倍

具体可以参考pretrain这个系列的文章。

## 如何优化BERT性能和效果

一般遇到这个问题，我们从三方面进行改进。

- 数据层面
  - 更大的数据量，更好的数据质量
  - 数据预处理，word-piece等。
  - 知识增强：Ernie/也有模型部分的。
- 任务设计
  - SOP任务
  - 位置预测任务
  - MLM任务变化
- 模型优化
  - 参数修改
    - 层数
    - 组件数（multi-head）
    - input替换
  - 某些重要组件优化或者替换
    - 替换优化核函数
    - 参数共享，矩阵分解
  - 模型压缩
    - fastbert
    - distillbert
    - tinybert等
- 设计变化：
  - 自编码 -> 自回归等

## ALBERT的参数共享到底是什么原理

一般来讲，共享参数，降低参数里边，效果最明显，作用最大的其实就是全联接层。例如在CNN中的卷积核理论上也是有参数共享的含义。意思是，底层多个值共用一个kernel，从而显著减小参数量和计算量。

albert里面也是一样的，那在哪里可以降低全联接层呢？ 很显然，是FFN和self-attention。 albert也是将这两个共享出去了。原理也是一样的，多层输出共享的是同一个“kernel”

