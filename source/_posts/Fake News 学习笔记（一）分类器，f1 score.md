---
title: Fake News 学习笔记（一）分类器，f1 score
tags: [NLP, Python]
originContent: ''
categories: [FakeNews]
toc: false
date: 2019-07-20 18:28:57
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---

## NFL data
美国职业橄榄球大联盟（National Football League，简称NFL）<!--more-->
你妹的，这教授我佛了。

## 逻辑回归
在第一节课中，x 轴代表该队触地得分的次数，y 轴代表该队是否胜利，输为 0，赢为 1，不使用线性回归的原因是该结果的输出为二分类问题，不需要数据的连续性，只需要输出 0 和 1 即可，同时该问题不是简单地线性问题，即很难用一条直线直接模拟该队触地得分次数与输赢的关系，同时受离群值的影响（当 x 值特别大，超过了正常的范围，就会影响正常值的分类），使用线性回归有很大的几率分类错误，所以所以使用逻辑回归方法进行分类。

逻辑回归使用 Sigmoid 函数，将函数的输入范围是负无穷到正无穷的定义域规定为 0-1 之内的范围，这样就解决了由于离群值对于阈值的影响作用。

## 参数定义（用于 F1 score 计算）
**True Positives (TP)**: Correct positive predictions
预测 yes，真实 yes
**False Positives (FP)**: Incorrect positive predictions (false alarm)
预测 yes，真实 no
**True Negatives (TN)**: Correct negative predictions
预测 no，真实 no
**False Negatives (FN)**: Incorrect negative predictions (a miss)
预测 no，真实 yes

## F1 score
用来衡量二分类模型精确的一种指标
**精确率**：TP/所有预测的 yes（所有预测 yes 中真实为 yes 的比率）
**召回率**：TP/所有真实的 yes（所有真实 yes 中预测为 yes 的比率）
**F1 score**：2/(1/P+1/C)=2TP(2TP+FN+FP)

## 决策树
利用 if-then 原则，按照树状结构的特点，叶节点表示其分类标记，非叶节点表示其各个 feature，利用 feature 进行匹配，直至找到最符合数据的分类。

## 随机森林（Random Forests）
包含多个决策树的分类器， 并且其输出的类别是由个别树输出的类别的众数而定。，随机森林对回归的结果在内部是取得平均

在得到森林之后，当有一个新的输入样本进入的时候，就让森林中的每一棵决策树分别进行一下判断，看看这个样本应该属于哪一类（对于分类算法），然后看看哪一类被选择最多，就预测这个样本为那一类。

## XGBoost
XGBoost是boosting算法的其中一种。Boosting算法的思想是将许多弱分类器集成在一起形成一个强分类器。因为XGBoost是一种提升树模型，所以它是将许多树模型集成在一起，形成一个很强的分类器。而所用到的树模型则是CART回归树模型。