---
title: 分布式计算学习笔记(二) Ray —远程对象与远程函数
date: 2019-11-20 11:01:35
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
recommend: 0
top: 100
toc: true
categories: [Distributed System]
---
# 概述
本篇博客主要介绍 Ray 远程对象，远程函数以及 Ray 进程之间并行的实现。
<!--more-->

# 前言
下学期去新加坡做毕设，老师给我订的主题是关于Ray-分布式执行框架的内容，其实就是想让我在这个框架中做一些应用，也可以说是大众化？前几天和HUST的挂名老师聊了聊，她也没有听说过这个框架，在网上搜了一下，说让我尝试一下在这个分布式执行框架中实现一个聚类算法，关键词有 Ray Tensor Clustering, 说实话，不懂，真的，看一个名次就会蹦出5个之前没见过的，多个名字叠加直接把我搞懵逼了。所以这个系列也算是我的学习笔记吧。
## Ray 概述
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
### 系统架构
![structure][image-1]
由图可知，Ray的结构基本符合 `master-workers` 的工作方式，其中每一个 `slave` 可以创建多个 `workers` 并行工作，并且在同一个节点中，`workers` 有可以共享的内存空间。

在每个节点中，存在一个 **ObjectStore**，用来存储只读数据对象，Worker可以通过共享内存的方式访问这些对象数据，这样可以有效地减少内存拷贝和对象序列化成本。ObjectStore底层由Apache Arrow实现。同时每个节点存在一个 **Plasma** 用来管理 **ObjectStore**。它可以在Worker访问本地ObjectStore上不存在的远程数据对象时，主动拉取其它Slave上的对象数据到当前机器。

在终端中运行 `ray.init(include_webui=True)`之后，会在本地创建 Ray集群环境，打开可视化界面如下。
![dashboard][image-2]
由图可知，本地共创建了1个节点，该节点共有16个 `workers` 进行工作

### 远程对象 - 不可变
远程对象存储在对象存储总，并利用唯一的对象ID进行引用。
ray.put() 和 ray.get() : 用过 python 对象和对象ID的转换
`x_id=ray.put(x)`：x为 python 对象，其函数返回值为该对象的对象ID ，数据结构为对象id的列表
`x=ray.get(x_id)`：x\_id\_为 对象ID，其函数返回值为该对象ID所对应的python对象 

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

第二个函数（远程函数）中，在调用之后会立即创建一个任务并分配给某一节点上的worker进行异步处理（由系统统一调度）。远程函数的输入参数可以通过值或者对象ID传入，函数返回结果为运算结果的唯一对象 ID。在实际情况汇总，一个远程函数可以返回多个对象ID。简单的异步执行的例子：

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

### **使用 ray.wait() 加快进程间的资源等待问题**
``` python
import time 
import random 
import ray 

ray.init(num_cpus = 4) 

@ray.remote 
def do_some_work(x): 
    time.sleep(random.uniform(0, 4))
    return x 

def process_results(results): 
    sum = 0 
    for x in results: 
        time.sleep(1)
        sum += x 
    return sum 

start = time.time() 
data_list = ray.get([do_some_work.remote(x) for x in range(4)]) 
sum = process_results(data_list) 
print("duration =", time.time() - start, "\nresult = ", sum) 
```

`data_list` 调用了4个远程函数进行执行，每个函数之间并行执行，最长时间为4s，之后再统一进行 sum 工作，所以时间等于 `4s + time(sum)`

为节省时间，利用 `ray.wait()` 函数进行处理，因为远程函数在调用的时候，会直接返回处理数据所对应的数据ID，即使该ID所对应的数据对象还没有返回，利用这个特性，加上 `ray.wait()` , 可以完成效率上的巨大提升。

``` python
import time 
import random 
import ray 

ray.init(num_cpus = 4) 

@ray.remote 
def do_some_work(x): 
    time.sleep(random.uniform(0, 4)) 
    return x 

def process_incremental(sum, result): 
    time.sleep(1)
    return sum + result 

start = time.time() 
result_ids = [do_some_work.remote(x) for x in range(4)] 
sum = 0 

while len(result_ids): 
    done_id, result_ids = ray.wait(result_ids) 
    sum = process_incremental(sum, ray.get(done_id[0])) 
print("duration =", time.time() - start, "\nresult = ", sum) 
```

在循环中，`ray.wait()` 返回了计算完成的id和还没有完成的id，将完成的id进行函数的计算工作，没有完成的作为循环判断条件继续进行处理，直至所有的任务都已完成。

![ray.wait()][image-3]
**问题：** 为什么每个都是 `done_id[0]` ，难道 `result_ids` 可以完成对 `done_id` 的某种判断还是像队列一样每次扔掉一个。
# Reference
- [https://blog.csdn.net/lzc4869/article/details/94663616][1]
- [https://blog.csdn.net/weixin\_43255962/article/details/88689665][2]
- [http://www.oreilly.com.cn/ideas/?p=2156][3]
- [https://www.cnblogs.com/fanzhidongyzby/p/7901139.html][4]

[1]:	https://blog.csdn.net/lzc4869/article/details/94663616
[2]:	https://blog.csdn.net/weixin_43255962/article/details/88689665
[3]:	http://www.oreilly.com.cn/ideas/?p=2156
[4]:	https://www.cnblogs.com/fanzhidongyzby/p/7901139.html

[image-1]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-str1.png
[image-2]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-str2.png
[image-3]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-wait.png