---
title: 分布式计算学习笔记(三) Ray —API & Actors
date: 2019-11-21 11:01:35
categories: [Distributed System]
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
recommend: 0
top: 100
toc: true
---

# 概述
本篇博客主要介绍 Ray API 简单的使用（基于actor和worker），以及运行一个简单的利用 Ray 进行修改的传统神经网络模型(MNIST)
<!--more-->
Actor: 有状态的 worker，当实例化一个新actor时，将创建一个新worker，并将acto的方法调度到该特定worker上，并且可以访问该worker并更改其状态。

CSDN: [Ray入门指南（3）----Ray API][1]

可以简单的理解为，在函数上加入 `@ray.remote`之后，这个函数就是 `worker`，在类上加入`@ray.remote`之后，这个类就是actor(有状态的workers)
``` python
	import os
	import ray
	import tensorflow.compat.v1 as tf
	tf.disable_v2_behavior()
	from tensorflow.examples.tutorials.mnist import input_data
	
	ray.init(num_gpus=8)
	# consruct neural network
	
	
	def construct_network():
	    # [None, 784]: data structure. total number of attribute is 28*28=784 with uncertain row number (batch size, can be of any size.)
	    x = tf.placeholder(tf.float32, [None, 784])
	    # total number of attribute is 10 (0-9) with uncertain row number
	    y_ = tf.placeholder(tf.float32, [None, 10])
	    W = tf.Variable(tf.zeros([784, 10]))  # Weights
	    b = tf.Variable(tf.zeros([10]))  # biase
	    y = tf.nn.softmax(tf.matmul(x, W) + b) # y = wx + b
	    # y_: real，y: prediction
	    cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ *
	                                                  tf.log(y), reduction_indices=[1]))  # loss function
	    # Use gradientdescentoptimizer to min the loss function
	    train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
	
	    # 1:search by row. tf.equal: 对比这两个矩阵或者向量的相等的元素，如果是相等的那就返回True，反正返回False
	    correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
	    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32)) # tf.cast： convert correct_prediction to float32
	
	    return x, y_, train_step, accuracy
	
	# define actor for structure
	@ray.remote(num_gpus=1)  # actor gpu数量为1
	class NeuralNetOnGPU(object):
	    def __init__(self, mnist_data):
	        self.mnist = mnist_data
	        # Set an environment variable to tell TensorFlow which GPUs to use. Note
	        # that this must be done before the call to tf.Session.
	        os.environ["CUDA_VISIBLE_DEVICES"] = ",".join(
	            [str(i) for i in ray.get_gpu_ids()])
	        with tf.Graph().as_default():
	            with tf.device("/gpu:0"):
	                self.x, self.y_, self.train_step, self.accuracy = construct_network()
	                # Allow this to run on CPUs if there aren't any GPUs.
	                config = tf.ConfigProto(allow_soft_placement=True)
	                #### normal network
	                # init = tf.initialize_all_variables()
	                # sess = tf.Session()
	                # sess.run(init)
	                ####
	                self.sess = tf.Session(config=config)
	                # Initialize the network.
	                init = tf.global_variables_initializer()
	                self.sess.run(init)
	
	    def train(self, num_steps):
	        for i in range(num_steps):
	            # load dataset by batch
	            batch_xs, batch_ys = self.mnist.train.next_batch(100)
	            # train
	            self.sess.run(self.train_step, feed_dict={
	                          self.x: batch_xs, self.y_: batch_ys})
	            if (i% 50):
	                print(self.get_accuracy())
	    def get_accuracy(self):
	        return self.sess.run(self.accuracy, feed_dict={self.x: self.mnist.test.images,
	                                                       self.y_: self.mnist.test.labels})
	
	
	# load MNIST dataset，并告诉Ray如何序列化定制类。
	mnist = input_data.read_data_sets("MNIST_data", one_hot=True)
	
	# Create the actor. 实例化actor并运行构造函数
	nn = NeuralNetOnGPU.remote(mnist)
	
	# Run a few steps of training and print the accuracy.
	nn.train.remote(200)
	accuracy = ray.get(nn.get_accuracy.remote()) # ray.get 从对象ID中进行数据的读取（python对象）
	print("Accuracy is {}.".format(accuracy))
```

相较于传统的神经网络结构，通过使用 Ray 进行了网络的封装，利用 Api 的形式进行网络调用，`construt_network()` 函数中定义了输入变量，权重，偏执，通过 softmax 隐藏层输出结果，同时利用交叉熵定义了损失函数，最终函数返回 输入，预测的结果，训练步数以及准确率。

在定义 Ray actor（类）中，进行了网络结构的初始化，训练的工作。这些函数的实现和没有使用 Ray 没有太大的差别，可以理解为进行了更为高级的封装便于 Ray 分布式框架的实现

Output:

```
(pid=20686) 0.9017
(pid=20686) 0.907
(pid=20686) 0.9047
(pid=20686) 0.9047
(pid=20686) 0.9025
Accuracy is 0.9049000144004822.
(pid=20686) 0.9026
(pid=20686) 0.9059
(pid=20686) 0.9088
(pid=20686) 0.9067
(pid=20686) 0.9049
```

在训练过程中，每50步进行结果的输出，其结果输出到终端，有图中可以得知，由于多进程工作，而只有一个位置进行结果的输出，难免会有优先抢断的问题，所以本应是最后输出的 Accuracy 放在了较为靠前的位置。

Q&A: 
1. python 中 `__init__(self)` 以及类的使用 
	`_init_(self)` 可以理解为JAVA的构造函数
	实例化类的时候会最先调用构造函数  

2. python class中不同函数的调用以及self的使用
``` python
	class MyClass:
		def __init__(self):
	    	pass
		def func1(self):
	    	# do something
	    	print('a')   #for example      
	    	self.common_func()
		 def func2(self):
	    	# do something
	    	self.common_func() # self表示类的实例
	    	 
		 def common_func(self):
			pass
```

3. Tensorflow 的基本定义以及使用
	`input-> hidden layer -> output layer -> 梯度下降进行参数的训练`
	用优化器 (optimizer) 减小误差
	`sess.run(Weights)`: sess类似于指针，用sess.run 指向网络结构中的 Weights，进行输出
	`placeholder`: 在session run的时候再进行数据的传入，可以理解为占一个位置
	只要是通过 `placeholder` 进行运算的东西，在sess.run里面都需要进行定义相关运算的参数
	```
	feed_dict={xs:x_data, ys:y_data}
	```
	`cross_entropy`: 交叉熵，用于目标与预测值之间的差距

[1]:	https://blog.csdn.net/weixin_43255962/article/details/88850456