---
layout: post
title: 为什么attention这么牛逼
date: 2021-03-10
Author: 乌卡 
tags: [attention, 注意力机制, 面试]
comments: true

---

这篇文章主要探讨attention机制的一些问题。attention是从图像领域中发展而来，逐步成为nlp领域最闪亮的明星。一个不懂得attention机制的nlper，根本不是一个nlper

## 什么是Attention

Attention 机制最早起源于图像领域，其思想萌发于90年代，在深度学习大行其道的时候，google团队用attention机制进行图像分类，获得了很好的性能，之后一发不可收拾。后来在机器翻译领域也逐渐采用了attention机制。之后的发展都是围绕着seq2seq模型展开的，自2017年的《Attention is all you need》之后，采用了self-attention机制，也就是transformer这个模型，之后慢慢将NLP带入了另一个领域

## Attention 的本质

Attention起源于人类视觉注意力系统中的关注点，其本质是对于重点问题的重点关注。所以他表现为对于不同特征点的注意力权值分配情况。[attention的本质](https://zhuanlan.zhihu.com/p/91839581)

![img](https://tva1.sinaimg.cn/large/008eGmZEly1gofxyy94o9j315o0hrgma.jpg)

## seq2seq中的attention

attention机制在这个时候是解决模型长时依赖问题的主要手段。其实就是在encoder - decoder上面加一个权值分配的向量。例如在翻译的领域，“我爱你”这三个字，“我”会更加关注“I”，那么在attention的权重中，“我”和“I”之间的权重是最大的。当然，其他的词也会分配一些权重，但是相比而言是非常小的。随着训练和数据的增加，其权值变化也会逐步趋近于稳定。

如图所示：

![image-20210311142017860](https://tva1.sinaimg.cn/large/008eGmZEly1gofxzu71knj30j20aywgw.jpg)

这个$c_1, c_2, c_3$就是的权值。

$C_i = \sum_{j=0}^{L_x}a_{ij}f(x_j)$

这个函数$f{(x_j)}$就是各种变换，用于计算权重的方法，那么很自然的，我们很容易猜到几个权值分配的方法（concatenate, dot, 矩阵相乘，相加，甚至softmax之类的）

## attention的标准范式

抛开整体的encoder-decoder模型，我们来看attention的标准范式。实际上attention是一个注意力分配洗漱的，就是通过输入的query，来查和他最相似的query之后查到他的value值。也就是标准的Q-K-V。这个范式非常常见，广泛应用于搜索系统、推荐系统、对话系统等等。

![image-20210311142736208](https://tva1.sinaimg.cn/large/008eGmZEly1gofyo1jf5dj30ip087jtp.jpg)

![image-20210311142756670](https://tva1.sinaimg.cn/large/008eGmZEly1gofyo0qnnmj30hg0f4dkk.jpg)

**那么他是怎么工作的呢**

- Query，Key计算得到相似度(QQ匹配)
- 权值归一化
- 同V加权求和

这个在我们的self-attention里面是应用最为标准的。当然在seq2seq中，他只是隐形的表示了，整体范式仍然是上面的方式。

#### 那么这个QKV到底怎么体现

[参考文章](https://zhuanlan.zhihu.com/p/59698165)

- 对于机器翻译而言，常见的是： Query 就是上一时刻 Decoder 的输出 ![[公式]](https://www.zhihu.com/equation?tex=S_%7Bi-1%7D) ， 而Key，Value 是一样的，指的是 Encoder 中每个单词的上下文表示。
- 对于英语高考阅读理解而言， Query 可以是**问题的表示**，也可以是**问题+选项的表示**， 而对于Key， Value而言，往往也都是一样的，都指的是**文章**。而此时Attention的目的就是找出**文章**中与**问题**或**问题+选项**的相关片段，以此来判断我们的问题是否为正确的选项。

此我们可以看出， Attention 中 Query， Key， Value 的定义都是很灵活的，不同的定义可能产生不一样的化学效果，比如 **Self-Attention** ，下面我就好好探讨探讨这一牛逼思想。

## Attention的优势

我们学习了很长时间的语言。实际上语言模型是有很多种的。例如，统计语言模型和分布式语言模型。我们目前在深度学习的时代，最主要还是走分布式语言模型的。这个模型强调一个词在上下文中的语义关系。

但是长久以来，基于循环神经网络、n-gram类型（fasttext, textcnn）等，很难解决长时依赖问题。即便是Bi-LSTM对这个问题有所缓解，但是并没有去解决。而attention机制提供了一个全局的注意力视野。可以让词和词忽略长时依赖直接建模。这也是attention的魅力所在。

总结下来，attention的优势主要是：

1. 一步到位的全局联系捕捉，效果很好
2. 支持并行，减少了模型训练时间
3. 模型复杂度低，参数比较小

## Attention的缺点

缺点是，他是实际上是忽略语序的，这是由于它是双向的结构。但实际上，很多论文也基于这个尽心改进。首先，transformer中提出了相对位置编码，解决了相对语序的问题。而进一步的，后续文章为了解决前后顺序的问题，提出了复杂位置编码（用复数表示），有兴趣的朋友可以去看看。

## Attention的类型

[参考文章](https://zhuanlan.zhihu.com/p/35739040)

### 权值计算方法

#### 计算形式

- Soft-Attention：这是比较常见的Attention方式，对所有key求权重概率，每个key都有一个对应的权重，是一种全局的计算方式（也可以叫Global Attention）。这种方式比较理性，参考了所有key的内容，再进行加权。但是计算量可能会比较大一些。表现形式就是嫁一个softmax
- Hard-Attention：这种方式是直接精准定位到某个key，其余key就都不管了，相当于这个key的概率是1，其余key的概率全部是0。因此这种对齐方式要求很高，要求一步到位，如果没有正确对齐，会带来很大的影响。另一方面，因为不可导，一般需要用强化学习的方法进行训练。（或者使用gumbel softmax之类的）
- Local-Attention：这种方式其实是以上两种方式的一个折中，对一个窗口区域进行计算。先用Hard方式定位到某个地方，以这个点为中心可以得到一个窗口区域，在这个小区域内用Soft方式来算Attention。

#### 所用信息

假设我们要对一段原文计算Attention，这里原文指的是我们要做attention的文本，那么所用信息包括内部信息和外部信息，内部信息指的是原文本身的信息，而外部信息指的是除原文以外的额外信息。

- General Attention: 这种方式利用到了外部信息，常用于需要构建两段文本关系的任务，query一般包含了额外信息，根据外部query对原文进行对齐。(有点带着预训练的词向量/句向量的意思)
- Local Attention：这种方式只使用内部信息，key和value以及query只和输入原文有关，在self attention中，key=value=query。既然没有外部信息，那么在原文中的每个词可以跟该句子中的所有词进行Attention计算，相当于寻找原文内部的关系。

#### 层次结构

- 单层attention
- 多层attention
- 多头attention

## 相似度计算方式

- dot
- 矩阵相乘
- cos相似度
- concatenate
- 感知机

