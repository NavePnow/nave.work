---
title: 痛苦的MARL(二)
date: 2019-07-04 20:26:52
tags: [RL, Python]
categories: [MARL]
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---
# 多智能体强化学习(二) 基础算法（MiniMax-Q，NashQ，FFQ，WoLF-PHC）
<!--more-->
## 1. 引言
再多智能体强化学习中，需要多 agent 在与环境交互过程中不断学习每个状态的奖励值 Q函数，再通过 Q函数来学习得到最优纳什策略。
**合理性**（rationality）是指在对手使用一个恒定策略的情况下，当前智能体能够学习并收敛到一个相对于对手策略的最优策略。

**收敛性**（convergence）是指在其他智能体也使用学习算法时，当前智能体能够学习并收敛到一个稳定的策略。通常情况下，收敛性针对系统中的所有的智能体使用相同的学习算法。

## 2. Minimax-Q算法
**Condition**: 两个玩家的零和随机博弈(eg.抛硬币)
**Idea**: 利用 Qlearning 的方法更新Q值
**Algorithm**:
![avatar](https://pic2.zhimg.com/80/v2-e907b291d3ec2e6dd62c96a86b4fd171_hd.jpg)

## 3. Nash Q-Learning算法
**Condition**: 多个玩家的一般和博弈(完全对抗博弈、完全合作博弈以及二者的混合博弈)
**Idea**: 使用二次规划求解纳什均衡点
**Algorithm**:
![avatar](https://pic4.zhimg.com/v2-5d50dc1f2ad874c22f10d5e796f64347_r.jpg)
通过算法可以得知，在更新 Q 值时，取代了传统的期望奖励 V 函数，这里用了 Nash 函数进行了更新,需要观测其他所有智能体的动作 $a_i$ 与奖励值 $r_i$

## 4. Friend-or-Foe Q-Learning算法
**Condition**: 一个智能体i，将其他所有智能体分为两组，一组为i的friend帮助i一起最大化其奖励回报，另一组为i的foe对抗i并降低i的奖励回报，因此对每个智能体而言都有两组。这样一个n智能体的一般和博弈就转化为了一个两智能体的零和博弈
**Idea**: 将剩余 agent 进行分组，在纳什均衡策略求解中，传统的所有 action 替换为了帮助自己的 action 和敌人的 action
Algorithm:
![avatar](https://pic1.zhimg.com/80/v2-2f4cff250568ca25853f1bc8e23e3b54_hd.jpg)
为了更新 Q 值，每个智能体需要在每一步观测其他所有friend与foe的执行动作。

## 5. WoLF Policy Hill-Climbing算法
**Condition**: 每个智能体只用保存自己的动作来完成学习任务,个人认为为了避免前面三个算法带来的维度灾难问题，因为每个 agent 都需要存储所有 agent 的动作集。
**Idea**: 
**WolF**:当智能体做的比期望值好的时候小心缓慢的调整参数，当智能体做的比期望值差的时候，加快步伐调整参数。
**PHC**:一种单智能体在稳定环境下的一种学习算法。该算法的核心就是通常强化学习的思想，增大能够得到最大累积期望的动作的选取概率。该算法具有合理性，能够收敛到最优策略。其算法流程如下
**PHC Algorithm**:
![avatar](https://pic4.zhimg.com/80/v2-8420ec197cb0516076725645a1359ab3_hd.jpg)
**WOLF+PHC Algorithm**:
通过PHC算法进行学习改进策略,收敛性没有得到证明。
![avatar](https://pic2.zhimg.com/80/v2-a549b9cfd895898e7a4cc74d7432ce55_hd.jpg)
