---
title: 分布式计算学习笔记(二) Ray —远程对象与远程函数
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

## ray分布式框架的介绍

### 远程对象 - 不可变
远程对象存储在对象存储总，并利用唯一的对象ID进行引用。
ray.put() 和 ray.get() : 用过 python 对象和对象ID的转换
`x\_id=ray.put(x)`：x为 python 对象，其函数返回值为该对象的对象ID ，数据结构为对象id的列表
`x=ray.get(x_id)`：x_id_为 对象ID，其函数返回值为该对象ID所对应的python对象 

``` python
		result_ids = [ray.put(i) for i in range(10)]
		result_ids[0]
		ray.get(result_ids[0]) # 0
		ray.get(result_ids)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 远程函数

``` python
		# Normal function
		def add1(a, b):
		return a + b
	
		# @: Decorator
		@ray.remote
		def add2(a, b):
		return a + b
	
		x_id = add2.remote(1, 2)
		ray.get(x_id)  # 3
```

第二段个函数（远程函数）中，在调用之后会立即创建一个任务并分配给某一节点上的worker进行异步处理（由系统统一调度）。远程函数的输入参数可以通过值或者对象ID传入，函数返回结果为运算结果的唯一对象 ID。在实际情况汇总，一个远程函数可以返回多个对象ID。简单的异步执行的例子：

``` python
	import time
	
	def f1():
	    time.sleep(1)
	
	@ray.remote
	def f2():
	    time.sleep(1)
	
	# 这个操作需要10秒.
	[f1() for _ in range(10)]
	
	# 下面的操作只需要一秒钟(假设系统至少有10个cpu核心)
	# 在Google Colab中由于服务器的CPU支持超线程技术，下面的操作只使用了5s（单核CPU）
	ray.get([f2.remote() for _ in range(10)])
```

远程函数输入与返回的例子：

``` python
	add2.remote(1, 2)
	add2.remote(1, ray.put(2)) # 系统将从对象存储中检索相应的对象
	add2.remote(ray.put(1), ray.put(2))
	
	@ray.remote(num_return_vals=3)
	def return_multiple():
	        return 1, 2, 3
	a_id, b_id, c_id = return_multiple.remote()
```

任务间的依赖关系：

``` python
	import numpy as np
	
	@ray.remote
	def generate_data():
	    return np.random.normal(size=1000)
	
	@ray.remote
	def aggregate_data(x, y):
	    return x + y
	
	# 生成一些随机数据。这将启动100个任务，这些任务将在多个节点上并行执行，
	# 结果数据将分布在集群的各个节点中（此处假设是在使用ray的分布式集群上使用的）。
	# 如果是在一台多核电脑上运行，则会根据核心数进行确定并行的数量。
	# 此时date的ID内存中有100*1000个数据
	# data 是100个对象ID的list，每一个对象ID所对应的python对象中共有1000个数据
	
	data = [generate_data.remote() for _ in range(100)]
	print(len(ray.get(data))) # 100
	
	# 执行树缩减，在累积相加过程中，取出两个对象ID所对应的python数据，想对应的数据进行相加，直到只剩下一个数据对象
	while len(data) > 1:
	    data.append(aggregate_data.remote(data.pop(0), data.pop(0)))
	
	#获取结果 1000个数据
	ray.get(data) # 1
```
# Reference
- [https://blog.csdn.net/lzc4869/article/details/94663616][1]
- [https://blog.csdn.net/weixin\_43255962/article/details/88689665][2]

[1]:	https://blog.csdn.net/lzc4869/article/details/94663616
[2]:	https://blog.csdn.net/weixin_43255962/article/details/88689665