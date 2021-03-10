---
layout: post
title: 【收藏】优秀博文收录
date: 2021-03-05
Author: 乌卡 
tags: [综述, 论文解读, 阅读]
comments: true
---

本篇收录一些比较优秀的文章，可供随时翻看

## 论文及解读

### NLP相关

1. [预训练模型超全知识点梳理与面试必备高频FAQ](https://mp.weixin.qq.com/s/rYNW36Iztz0Bh7pBX9pdrg) 

   - 简介： 这篇文章实际上是一个对于《Pre-trained Models for Natural Language Processing: A Survey》的解读和二次加工。没有时间看原论文的可以先看看这个。

   - 关键词：综述，论文解读

2. [NLP中预训练模型的综述-1深度长文-慎点](https://zhuanlan.zhihu.com/p/353054197)， [NLP中预训练模型的综述-2深度长文-慎点](https://zhuanlan.zhihu.com/p/353638297)
   - 简介：这篇文章是对于《Pre-trained Models for Natural Language Processing: A Survey》的解读和二次加工，比上面那个更详细
   - 关键词：综述，论文解读
3. [Attention Is All You Need 中文版](https://blog.csdn.net/nocml/article/details/103082600)
   - 简介：这个是Attention is all you need的翻译版本，翻译质量确实还比较好。可以配合原文一起食用
   - 关键词：transformer
4. [各种transformer的改进](https://zhuanlan.zhihu.com/p/271198097)
   - 简介：transformer有很多缺点，例如：固定长度，self-attention机制计算太过复杂。改进方案有：
     - Transformer-XL:处理超长文本。实际上是加了一个隐藏层$h_r$，用来处理$r$层的状态，讲这个状态加载到新的计算例
     - Adaptive Attention Span：动态的调整attention的窗口的大小。主要想法是：1. 计算token的attention值时，不是序列中每个token都有效；2.对于multi-head attention，对于每个head， token的attention关注的重点（pattern）也不同。针对上述两点，他设计了一个函数$m_{z}(x)=\operatorname{clamp}\left(\frac{1}{R}(R+z-x), 0,1\right)$附加到不同的头上，用来学习不同头的权重。
     - Reformer: 局部hash敏感。这是一个比较工程化的方法，但是放在attention这里有奇效。局部敏感哈希会将相似的Item根据哈希值分在同样的桶内（bucket）中，从而利用一个桶的items的计算近似代替真正的计算值。同样的，attention的实现也可以像上述的那样。先token进行哈希分桶，然后计算某个token的attention时只需考虑所在分桶的tokens就行了，从而大量的减少运算量。如何对token进行分桶呢？Reformer利用随机矩阵实现哈希。除了上面的局部敏感哈希以外，Reformer也通过将可逆转的残差网络（Reversible Residual Network）应用到attention的计算中。神经网络反向传播计算梯度时需要用到每一层的激活值，这些激活值占了一大部分GPU的内存。
     - Linformer：修改self-attention的核函数，使其变成线性运算。矩阵变化，使得$n\times n$ 变成$n \times k$，当$n >> k$ 时，计算量大幅下降。
   - 关键词：transformer优化
5. [transformer问题整理](https://zhuanlan.zhihu.com/p/266695736)
   - 简介：这篇收集比较偏面试向，可以多看看。
   - 关键词：transformer，面试



### 机器学习基础知识相关

1. [机器学习中的偏差和方差](https://blog.csdn.net/mingtian715/article/details/53789487)
   - 简介：主要介绍在机器学习中偏差和方差的由来。模型越复杂，其方差越大，偏差越小，对应过拟合。模型越简单，偏差越大，方差越小，对应欠拟合。例如KNN偏差大，神经网络偏差小。
   - 关键词：方差，偏差，机器学习
2. [选择偏差怎么解决](https://zhuanlan.zhihu.com/p/87006801)
   - 简介：在推荐系统、统计分析和数据分析领域经常会出现选择偏差的问题。选择偏差和幸存者偏差都属于在选择样本上不够全面造成的。例如，conv-19在巴西和土耳其接种试验中，二者最终的数据差异极大。幸存者偏差更是老生常谈，在此不做赘述。解决的方法很简单，应该深入到数据中，收集更能反映问题的数据。
   - 关键词：选择偏差，统计分析，数据分析0

## 参考文献

1.  [预训练模型超全知识点梳理与面试必备高频FAQ](https://mp.weixin.qq.com/s/rYNW36Iztz0Bh7pBX9pdrg)
2. [NLP中预训练模型的综述I深度长文-慎点](https://zhuanlan.zhihu.com/p/353054197)
3. [NLP中预训练模型的综述I深度长文-慎点](https://zhuanlan.zhihu.com/p/353638297)

