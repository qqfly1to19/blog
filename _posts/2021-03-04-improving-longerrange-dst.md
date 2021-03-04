---
layout: post
title: 提高长文本对话能力
date: 2021-03-02
Author: 乌卡 
tags: [长文本, DST, 对话系统, 论文阅读]
comments: true

---

不同于IM类/文字类的对话场景，在ASR对话场景中，长文本的对话是最为常见的，经统计，在我们的系统中，80%以上的对话都是长文本对话（超过32个字）。提高长文本的对话能力尤为关键。这篇2021年2月份谷歌团队发表新文章，着重讲述了这个事情

> 题目：IMPROVING LONGER-RANGE DIALOGUE STATE TRACKING
>
> arxiv: https://arxiv.org/pdf/2103.00109.pdf

## Abstract

DST是任务型对话中的一个重要组件，而目前大部分的DST系统在短文本会话中能够捕捉到可信的状态，但是在长文本会话中表现不好，这是由于无法没有良好的注意力机制。本文将集中精力去提升DST系统中的长文本问题。我们将这个问题分为三个方面：

- 设计一个层次槽位状态的模型
- 平衡通用场景和面向任务场景中的语言理解训练过程。
- 通过数据扰动(Data Perturbation)来增加长文本会话能力

在MultiWOZ基准上实验，并通过一组消融实验来证明上述组件的在长文本上的有效性。

## Introduction

DST在面向任务的对话系统中是一个非常重要的组件，DST系统的可信程度随着文本的长度增加而逐步降低。自2019年以来的NLP技术，尤其是基于PTM的下游微调任务和大体量的模型的发展十分迅猛。但是仍然很难恰当的解决长文本的多轮和多领域对话问题。例如，BERTs可以很好的解决段文本中的精确预测问题，但是很难在多轮中信息中获取正确的推理状态。

## Model

### Base Model

基线实际上就是Bert 模型，通过Bert 去产出类别和槽位。

当接收到一个新的对话，模型首先将历史所有的对话全部encode, 同样，还有一个encoder用于分类和槽位提取。同时，还有一个状态判断的，用于是否作答的分类：{active, don’t care, inactive}。 我们所有的值只需要关注需要active这个类别(在我们的场景中，由于评价的需要，非作答的我们也要关注)。

### HIERARCHICAL SLOT STATUS PREDICTION

我们认为，DST的预测准确性严重依赖于槽位的准确性。而且不同的评测标准和槽位推理会对结果产生很大的影响。

状态预测是困难的，因为大多数时隙在会话的任何一个回合都是不回复的，这给训练和推理过程都造成了严重的类不平衡问题。而且，随着文本变长，需要追踪的信息变多，准确率回极速下降。所以我们提出了一个分层槽位状态预测过程。

我们先预测当前轮次的domain，然后预测值预测需要回复的slot。这有助于让我们降低错误的概率。

![image-20210304153255579](https://tva1.sinaimg.cn/large/008eGmZEly1go7wr8vxxwj30n007l3zg.jpg)

![image-20210304153315128](https://tva1.sinaimg.cn/large/008eGmZEly1go7wrkz470j30i706xdgq.jpg)

![image-20210304152827415](https://tva1.sinaimg.cn/large/008eGmZEly1go7wmnqefpj30om0lsgor.jpg)



<font color=red>在这里要说明的是，这个方法实际上就是一个处理过程。总体来说比较简单，实际上就是去预测哪些该去回复。而且其实把slot绑定到了domain(intent)当中</font>

## Training Procedure

### Pre-training and fine-tune

#### Domain closeness

-  大规模预训练预料，Meena open-domain chat bot (Adiwardana et al., 2020)

#### continued MLM

- ToDBERT，  (Wu et al., 2020)

### Data Preturbation

加入扰乱信息，这个方法太好了。是一个不错的方法。

## Experiement

### 模型效果

![image-20210304154613548](https://tva1.sinaimg.cn/large/008eGmZEly1go7x5359y2j30om0ay75t.jpg)

- **TRADE**: A transferable dialogue state generator that generates dialogue states from utterances using a copy mechanism (Wu et al., 2019).
-  **DS-DST**: A dual strategy model that simultaneously handles non-categorical and categorical slots (Zhang et al., 2019).
-  **SST**: A graph attention network that predicts states from utterances and schema graphs containing slot relations in edges (Chen et al., 2020).
-  **Trippy**: A triple copy strategy where a slot is filled by one of the three copy mecha- nisms (Heck et al., 2020).
- **SimpleToD**: Training a simple language model that casts DST as a sequence prediction problem (Hosseini-Asl et al., 2020).

### 数据扰动

几种加入数据扰动的方法

- 随机加入一些干扰的utterance，通过ToDBert, MultiWOZ, 同义词等
- 加入的utterrance数据量：$N = \{2,3,4\}$
- 随机插入概率$p=\{0.2, 0.4, 0.6\}$

## Analysis

### 消融实验分析



![image-20210304155411371](https://tva1.sinaimg.cn/large/008eGmZEly1go7xddit5gj30kq0btq3m.jpg)

### 几个结论

- 毫无疑问，短文本会话的准确率高于长文本会话准确率
- 从最简单的Base模型到我们这个模型，长文本和短文本会话都有所提高，这都是分层slot的功效。
- 分层slot是最有效的方法

### 数据扰动实验

![image-20210304155832010](https://tva1.sinaimg.cn/large/008eGmZEly1go7xhwh28gj30rh0kh0vs.jpg)

## 讨论：为什么分层slot这么有效？

暂时没做讨论