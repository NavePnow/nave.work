---
title: 痛苦的MARL(三)
date: 2019-07-04 21:09:52
tags: [RL, Python]
categories: [MARL]
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---
# 多智能体强化学习入门（三）——MFMARL算法（Mean Field Multi-Agent RL）
<!--more-->
**Idea**:将传统的多智能体算法（每个智能体都需考虑其他所有智能体的动作以及状态得到联合动作值函数）替换成一种近似假设（其他所有智能体对其产生的作用可以用一个均值替代）--MFT 平均场理论
Based on MFT, there are two basic algoritms: MFQ & MFAC. （分别是对 Q learning 和 AC 算法的改进）
## 1. Mean Field MARL
**Idea**: 将 Q 函数中的参数调整为只包含邻居之间相互作用的形式：
$$
    Q_j(s,a)=\frac{1}{N_j}\sum _{k\epsilon N_{(j)}}Q_j(s,a_j,a_k)
$$
其中$N_j$表示邻居节点个数，状态信息 s 为全局信息。

### 1. Mean Field 近似




### 2. 算法设计
MF-Q + MF-AC Algorithm

#### 1. MF-Q




#### 2. MFAC
