---
title: 分布式计算学习笔记(二) Ray
date: 2019-11-21 11:01:35
categories:
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
recommend: 0
top: 100
toc: true
---
# Ray
## 前言
<!--more-->
下学期去新加坡做毕设，老师给我订的主题是关于Ray-分布式执行框架的内容，其实就是想让我在这个框架中做一些应用，也可以说是大众化？前几天和HUST的挂名老师聊了聊，她也没有听说过这个框架，在网上搜了一下，说让我尝试一下在这个分布式执行框架中实现一个聚类算法，关键词有 Ray Tensor Clustering, 说实话，不懂，真的，看一个名次就会蹦出5个之前没见过的，多个名字叠加直接把我搞懵逼了。所以这个系列也算是我的学习笔记吧。
## 概述
Ray是UC Berkeley RISELab新推出的高性能分布式执行框架，它使用了和传统分布式计算系统不一样的架构和对分布式计算的抽象方式，具有比Spark更优异的计算性能。
- 优点:
	- 海量任务调度能力。
	- 毫秒级别的延迟。
	- 异构任务的支持。
	- 任务拓扑图动态修改的能力。
- 缺点：
	- API层以上的部分还比较薄弱，Core模块核心逻辑估需要时间打磨。
	- 国内目前除了蚂蚁金服和RISELab有针对性的合作以外，关注程度还很低，没有实际的应用实例看到，整体来说还处于比较早期的框架构建阶段。
- 用途：  
	增强学习
	- 分类
	- 聚类
	- 图像识别
	- 推荐系统
	- 文本翻译
	- Application: deep reinforcement learning using RLlib, scalable hyperparameter search using Ray Tune, automatic program synthesis using AutoPandas, etc. (advanced library from tutorial)


# Reference
- [https://blog.csdn.net/lzc4869/article/details/94663616][1]
- 

[1]:	https://blog.csdn.net/lzc4869/article/details/94663616