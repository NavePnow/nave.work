---
title: Fake News 学习笔记（二） one-hot-coding Stem-and-Lem Word2Vec
tags: [NLP, Python]
categories: [FakeNews]
toc: false
date: 2019-07-30 17:14:16
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---

# One-hot coding(独热编码)
<!--more-->
果然又是 1个半小时只记住专有名字的课程。
**target**:将非数值类型量化数值类型，以便于模型的输入
**process**:N位状态寄存器来对N个状态进行编码就是将所有状态排列，具有哪些状态就将状态进行标记
**instance**:
face = ['handsome','ugly']
stature = ['tall','middle','short']
country = ['Chinese','American,'Japan','korea']
共有 9 种状态，用 9 位数字表示。
['handsome','tall','Japan'] 表示为 101000010
['ugly','short','Japan'] 表示为 010010010

# Bag of words
不考虑单词在文章中的顺序，只考虑单词在文章中的词频率(occurence)

# Stemming and Lemmatizing
**cliche**:normalize different forms of the same word to a single root token before indexing
stemming can often create non-existent words, whereas lemmas are actual words.

## Stemming
找词根(chops off the endings of different forms of words) "derivational affixes"
**questions**:some decorations like ir or un, some of them will be deleted? eg:unchange to change (not implentation)
## Lemmatizing
根据词典找单词本身的形式 eg: saw to see

# Word2Vec
**cliche**:Word2Vec使用一层神经网络将one-hot（独热编码）形式的词向量映射到分布式形式的词向量。使用了Hierarchical softmax， negative sampling等技巧进行训练速度上的优化.
在前文中介绍了 one-hot coding，但是由于英文单词词汇量巨大的特征，每一个单词都对应上文例子中的 feature，可想而知，会造成维度灾难和数据稀疏的问题（妈的，上课这个东西讲了近 40min），所以使用Word2Vec能够解决这些问题。

## TD-IDF
评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度，TF意思是词频(Term Frequency)，IDF意思是逆文本频率指数(Inverse Document Frequency)。
## softmax
把一些输入映射为0-1之间的实数，并且归一化保证和为1
## CBOW(Continous Bag of Words)
已知词w上下文context(w)前提下，预测当前词w
## skip-gram
已知当前词w，预测其上下文context(w)
## distributed representation
通过训练得到每个词k 维实数向量，通过词间距离来计算词间相似度。
通过训练，将每一个词映射到一个固定长度的短向量中，把词的信息分布到各个分量中，并且语义相近的词向量见距离越近
## 参数设置
size: Number of dimensions for the word embedding model
window: Number of context words to observe in each direction
min_count: Minimum frequency for words included in model
sg (Skip-Gram): '0' indicates CBOW model; '1' indicates Skip-Gram
alpha: Learning rate (initial); prevents model from over-correcting, enables finer tuning
iterations: Number of passes through dataset
batch_words: Number of words to sample from data during each pass