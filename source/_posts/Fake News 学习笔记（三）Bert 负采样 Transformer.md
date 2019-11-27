---
title: Fake News 学习笔记（三）Bert 负采样 Transformer
tags: [NLP, Python]
categories: [FakeNews]
toc: false
date: 2019-08-11 22:36:47
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---

# 使用 Google Bert 实现 Word-Embedding
<!--more-->
(由于组内分工，我和另外一位同学负责文字处理部分，所以这里主要介绍对于 Google Bert 的学习过程以及对于文字处理的一些细节)
**idea**:
1. 将 cleaned 后的文本进行提取，如果 len(text)为 0，删掉 label
2. 使用学习笔记二中提到的stemming lemmatizing技术，对文本进行处理(不知道结果怎么样)
3. to be continued

## 负采样
负采样通过使每一个训练样本仅仅改变一小部分的权重而不是所有权重。也就是对正确的一个输出单词up，选择5-20个不正确的单词low。在 Bert 中，首先给定的一个句子，下一句子正例（正确词），随机采样一句负例（随机采样词）,句子级上来做二分类（即判断句子是当前句子的下一句还是噪声），类似word2vec的单词级负采样。

## Transformer 模型结构
![The Transformer Architecture](https://i.loli.net/2019/08/11/DZd3vpK7OhyjgLX.png)

如图所示，左边为encoder，右边为 decoder，具有高并行性的特点抱歉，现阶段本人能力有限，根本看不懂，反正牛逼就完事了，放上 link，万一以后回看呢。
[Transformer arch.](https://www.cnblogs.com/sxron/p/10035802.html)
### **multi-head attention:**
将一个词的vector切分成h个维度，求attention相似度时每个h维度计算。由于单词映射在高维空间作为向量形式，每一维空间都可以学到不同的特征，相邻空间所学结果更相似，相较于全体空间放到一起对应更加合理。比如对于vector-size=512的词向量，取h=8，每64个空间做一个attention，学到结果更细化。
### **self-attention：**
每个词位的词都可以无视方向和距离，有机会直接和句子中的每个词encoding。比如下面这个句子，每个单词和同句其他单词之间都有一条边作为联系，边的颜色越深表明联系越强，而一般意义模糊的词语所连的边都比较深。比如：law，application，missing，opinion。。。
### **position encoding:**
因为transformer既没有RNN的recurrence也没有CNN的convolution，但序列顺序信息很重要，比如你欠我100万明天要还和我欠你100万明天要还的含义截然不同。。。
　transformer计算token的位置信息这里使用正弦波，类似模拟信号传播周期性变化。这样的循环函数可以一定程度上增加模型的泛化能力。
　　但BERT直接训练一个position embedding来保留位置信息，每个位置随机初始化一个向量，加入模型训练，最后就得到一个包含位置信息的embedding（简单粗暴。。），最后这个position embedding和word embedding的结合方式上，BERT选择直接拼接。
![Output](https://i.loli.net/2019/08/11/mnRFiSh8MkjHA5I.png)

## Google Bert
NLP任务分为两部分，其一是预训练产生**词向量**，其二是对词向量进行操作(下游 NLP 任务)

### 预训练产生词向量
相比于之前所用到的 word2vec，负采样从 word-level 升级到了 sentence-level ，从而可以获取句间关系。
Bert 使用双向encoding(模型在处理某一个词时，它能同时利用前面的词和后面的词两部分信息)，采用看不懂的 Transformer 结构，直接获得一整个句子的唯一向量表示。在 Transformer 结构中，最终的输入由下面 3 个embedding 组成。
![Input](https://i.loli.net/2019/08/11/jdZ792TAniEIPhY.jpg)
EA 表示左句子，EB 表示右句子，CLS为特殊标记符，供 Transformer 对 CLS进行深度 embedding

###预训练模型和加载的训练集之间的关系
使用预先训练的模型，该模型已经在大型数据集上进行了训练，然后针对特定任务进行微调。

### 下游 NLP 任务
在获得 Bert 词向量后，只需要在词向量上加入简单的分类器即可完成工作