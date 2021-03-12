---
layout: post
title: 为什么transformer这么牛逼？
date: 2021-03-05
Author: 乌卡 
tags: [机器学习, 评价方法, 面试]
comments: true

---

transformer可以认为是一种设计较为复杂的一种模型。其核心是self-attention，但是仍然有很多别的设计。例如input/output输出，layer， encoder/decoder， layer normalization，相对位置编码，残差网络等等。每个既是经验的总结也包含着设计者的智慧。本文就是从这些问题出发，试图找到一些规律。

## 1 为什么使用self-attention

### 论文上的解释

论文上主要是通过把self-attention和RNN，CNN进行对比。$input=(x_1, x_2, ..., x_n)$ 映射到相同长度的序列$output=(z_1,z_2,...,z_n)$

从三个指标上分析：

- 每一层的计算复杂度
- 能够并行的计算
- 长时依赖的长度有多长

![image-20210305144024148](https://tva1.sinaimg.cn/large/008eGmZEly1go90uy4srrj30it069410.jpg)

从这三个方面来看，self-attention确实优于RNN和CNN。这里的复杂度需要仔细去看。

## 2 Transformer结构是什么样子的

我们从整体到部分来仔细介绍一下transformer结构

### encoder & decoder

- encoder是由N=6个相同的大模块堆叠而成。每个大模块有两个子模块。即Multi-head Attentioni和FFN
  - 其中第一层实际上是接入input层的。其余的输入都是上一层encoder的输出
- decoder也是由N=6的大模块堆叠。其中包含了三个字模块。分别是Multi-head self-ttention、Multi encoder-decoder交互模块和FFN。
  - 同样，Decoder每一层的输入也是不一样的。其中第一层的输入在训练和测试不一样。每次训练接受的输入也是不一样，也就是所谓的shift-right(自回归网络)。其余的输入都是上一层的decoder输出。
  - 关于第一层。
    - 训练：每次的输入是上次的输入加上输入序列向后移一位的ground truth(就是新的单词再加上对应的embedding)。特殊的是，在第一次输入，应该是<BOS>，或者是<EOS>。其目标是预测下一个token是什么。实际中，往往是把目标序列的embedding统统输入，而在multi-head attention模块对序列进行mask
    - 在预测时，首先生成第一个位置的输出，然后再依次输入，直至预测结束。

### encoder端子模块

- Muti-head self-attention

  **将query和key-value键值对的一组集合映射到输出**，其中 query，keys，values和输出都是向量，其中 query和keys的维度均为 $d_k$，values的维度为$等下_v$ .公式如下：

  $ \operatorname{Attention}(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^{T}}{\sqrt{d_{k}}}\right) V$

  ![image-20210305154603307](https://tva1.sinaimg.cn/large/008eGmZEly1go93qoghq9j30e60gkaan.jpg)

- FFN

  这是一个标准的前馈神经网络。激活函数时Relu。

  $\operatorname{FFN}(x)=\max \left(0, x W_{1}+b_{1}\right) W_{2}+b_{2}$

  其中输入输出维度时512, 内层w维度时2048

### Decoder子模块

- Multi-head self-attention

  同encoder层中的一致，唯一不同的是。在decoder层的multi-head self-attention需要做musk。即将当前预测的token和之后的token全部musk掉。

- multi-head encoder-decoder attention交互模块

  形式上是一致的，唯一不同是Q,K,V的来源不同。其K和V的来源是encoder端的输出。目的是让Decoder端能够给予Encoder端更多的注意力。也就挂上了之前seq2seq中的attention机制了。

### FFN

同encoder端一致。

### 其他模块

- Add & Norm模块

  Residual + Layer Norm

- Position Encoding

  这是一种相对位置编码。相对位置编码是直接附加到encoder和decoder最底端的embedding输入的。他和你本身的embedding有着相同的维度。公式如下：

  $\begin{aligned}
  P E_{(p o s, 2 i)} &=\sin \left(\operatorname{pos} / 10000^{2 i / \mathrm{d}_{\text {model }}}\right) \\
  P E_{(p o s, 2 i+1)} &=\cos \left(\operatorname{pos} / 10000^{2 i / d_{\text {model }}}\right)
  \end{aligned}$

  其中$pos$是位置（0, 1, 2, 3）， $i$是维度。之所以选择这个函数，是因为三角函数的特性。

  $\begin{array}{l}
  \sin (\alpha+\beta)=\sin (\alpha) \cos (\beta)+\cos (\alpha) \sin (\beta) \\
  \cos (\alpha+\beta)=\cos (\alpha) \cos (\beta)-\sin (\alpha) \sin (\beta)
  \end{array}$

  Transformer中的Positional Encoding不是通过网络学习得来的，而是直接通过公式计算而来的，论文中也实验了利用网络学习Positional Encoding，发现结果与上述基本一致，但是论文中选择了正弦和余弦函数版本，因为三角公式不受序列长度的限制，也就是可以对比所遇到序列的更长的序列进行表示。

## 为什么Transformer强调self-Attention。

这是一种自注意力模型，实质是去学习自身的序列依赖关系。因为我们每次计算都是整个维度整体放进去计算，也让他具有忽略token之间的距离，直接计算依赖关系。从而学习到序列内部的结构。引入Self Attention后会更容易捕获句子中长距离的相互依赖的特征，因为**如果是RNN或者LSTM，需要依次序序列计算**，对于远距离的相互依赖的特征，要经过若干时间步步骤的信息累积才能将两者联系起来，而距离越远，有效捕获的可能性越小。

## 为什么使用Q K V，直接使用K V/Q V行不行

这里是存在争议的，有人认为是可以的。但是我认为在这个特殊结构里是不行的。这里不涉及到对核函数的改动。在Decoder里面Q的输入是decoder层的输出，而K和V是encoder层的输出。拆解不开。

而这种Q，K，V的方式，比较类似于搜索内部的QQ匹配和查询过程，比较符合直觉。

## Transformer为什么需要进行Multi操作，一个头行不行

如果不在乎效果，是可以的。但是将模型分为多个头，形成不同的子空间，在初始化以及之后的训练过程中，他们会**自己关注自己的不同方向信息**。最后再将信息综合起来。有点类似于**CNN的多层卷积核**。同时，有论文也对mutihead-attention进行了可视化，发现在训练中，大部分的头关注的都是基本相同的，但是总有那么一两个头跟别的头是不一样的。这也侧面证明了多头机制更具有泛化能力，更能发现内部的某些内在联系。

## Transformer相对于RNN/LSTM有什么优势

详细可参考上面的截图。即对比情况。主要是RNN/LSTM参数规模大，训练速度慢无法并行，和长时依赖的衰减问题。

## self-attention公式中的归一化有什么作用

首先要明确softmax的特性，他的特性就是随意做乘法和除法，都不会影响最终的值。而缩放是有明显好处的，可以防止函数推入到非常小的区域导致收敛困难。

## BERT中为什么用相对位置编码而其他位置编码方式

[参考这篇文章](https://blog.csdn.net/xixiaoyaoww/article/details/105459376)

### 目前由几种编码方式：

- 绝对位置编码(学习位置编码)。即从模型中学习到的位置。
- 相对位置编码，即attention中提到的。**位置的psotional encoding可以被位置线性表示，反应其相对位置关系。**
- 复杂位置编码。即不仅有位置还有方向。

### 优缺点分析

- 那么相对位置编码有啥缺点呢？

  从公示上来看，他是没有方向的。只能学习其相对的位置。为了更好的让模型捕获更精确的相对位置关系，比如相邻，前序（precedence）等，ICLR 2020发表的文章《Encoding Word Oder In Complex Embeddings》使用了复数域的连续函数来编码词在不同位置的表示。

- 那么从网络中进行学习难道不是更好么？

  - 而从论文中我们知道相对位置编码和网络中学习位置这两种方式的效果没有明显的差别。
  - 理解上看，网络中学习位置更加的简单直接，易于理解。从参数维度上，使用相对位置编码不会引入额外参数，而学习位置这种方式会增加的参数量会随线性增长而Complex Embedding在不做优化的情况下，会增加三倍word embedding的参数量。
  - 从可扩展性上看，学习中得到的位置可扩展性较差，只能表征在以内的位置，而另外两种方法没有这样的限制，可扩展性更强。(不太好理解)

## Bert和Transformer的贡献是什么

### Bert贡献



- 显示了双向预训练模型的重要性。Bert使用MLM是的模型得到了深度的双向表征能力
- 显示了预训练表征可以减少特殊任务的工程量。(pre-train model + specific-domain fine-tune)
- 打破了11项NLP任务记录

### Transformer贡献

- 没有采用RNN/LSTM这种大热的循环网络建模，使用了全新的multi-head self-attention,具有良好的效果
- 可以并行计算
- 长距离依赖问题

## Transformer为什么Q和K为什么不能点乘

按照相似度计算时可以的，点乘是一种计算相似度的方法。transofmer中为什么要舍近求远？因为点乘是要求两个向量是一模一样的，这样会导致在相似度计算时，自己和自己是最相似的。这种方式不利于其他向量对他的作用。如果QKV使用不同的权重矩阵初始化生成，，则保证在**不同空间进行投影，增强表达能力，提高泛化能力**

## 为什么在进行softmax之前需要对attention进行scaled

见上文：self-attention公式中的归一化有什么作用

## 为什么使用LN而不是BN

对于使用场景来说，BN在MLP和CNN上使用的效果都比较好，在RNN这种文本模型上使用的比较差。

BN在MLP中的应用： BN是对每个特征在batch_size上求的均值和方差，BN应用到NLP任务，相当于是在对默认了在同一个位置的单词对应的是同一种特征，

layer_norm针对的是文本的长度，整条序列的文本。

实际上，LN可以看作是对整句话进行norm，这个在直觉上是有意义的。而BN是同一个batch下的norm，而NLP任务的特殊性，不同样本之间方差比较大，将他们强行norm到同一个空间没有什么物理上的意义。

## transformer的物理解释是什么？

源自一篇微软研究院的文章：[Understanding and Improving Transformer From a Multi-Particle Dynamic System Point of View](https://arxiv.org/abs/1906.02762)

[博文参考](https://zhuanlan.zhihu.com/p/71747175)

- input问题。句子中的每一个单词里面每一个“星体”——粒子
- FFN部分：对应着物理中的对流——对流的背景场.F,建模粒子之间的关系
- multi-head self-attention部分：对应着物理中的扩散/物体间的相互关系（graph laplacian）——G建模对流的背景场。

也就是一个是粒子关系F，一个是对流场关系G。这个系统可以形式化为$\frac{d x}{d t}=F(x)+G(x)$

而transformer作为这样的一个解释

器，把G和F求解分开。先求一步G，再求一步F。物理上这个叫spliting scheme

当然，还有从量子力学角度去思考的。[CNM: An Interpretable Complex-valued Network for Matching](https://arxiv.org/abs/1904.05298)

