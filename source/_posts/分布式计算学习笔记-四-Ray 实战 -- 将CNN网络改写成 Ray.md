---
title: 分布式计算学习笔记(四) Ray 实战 -- 将CNN网络改写成 Ray
date: 2020-02-05 11:01:35
categories: [Distributed System]
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
recommend: 0
top: 100
toc: true
---

# 概述
本篇博客首先会对 CNN + MNIST 的神经网络结构进行分析（因为我也不会），之后利用 Ray 的 API进行改写，完成分布式框架上的深度学习工作。
<!--more-->
# 实现

## CNN 结构

### tf.reshape(tensor, shape, name=None)
将 tensor 变换为参数shape的形式
```
xs = tf.placeholder(tf.float32, [None,784])
x_image = =tf.reshape(xs,[-1,28,28,1])
```
其中，两个28表示了该图片的长宽是28个像素，厚度因为是黑白图片，所以为1，如果图片是rgb颜色，则为3，最开始的-1表示这种图片的多少，即 `x_image[0]` 为一张完整的图片。

### tf.nn.conv2d(input, filter, strides, padding, use\_cudnn\_on\_gpu=None, data\_format=None, name=None)
卷积操作，filter= patch(筛选器)
`input` : `[batch, in_height, in_width, in_channels]`
`filter`: `[filter_height, filter_width, in_channels, out_channels]`
其中 `in_cahnnels`代表的是图像的高度，输入为1，由于卷积操作之后图像的高度会变高，所以`out_cahnnels`要大于`in_channels`的值
`strides`: `[1, x_movement, y_movement, 1]` 第一个参数和第四个参数都是1，第二个和第三个是 x 轴和 y 轴的移动距离，如果都是2，代表着每次移动2个像素（会有一个像素的信息不能被采集）
`padding`: 当`padding=SAME` 时，输入与输出形状相同

### tf.nn.max\_pool(value, ksize, strides, padding, name=None)
池化操作
`value`: `[batch, in_height, in_width, in_channels]`池化输入参数
`ksize`:  `[1, height, width, 1]` 池化窗口大小
``strides` = `[1, x_movement, y_movement, 1]`` 第一个参数和第四个参数都是1，第二个和第三个是x轴和y轴的移动距离，如果都是2，代表着每次移动2个像素（会有一个像素的信息不能被采集）
`padding`: 当 `padding=SAME` 时，输入与输出形状相同

### 基于 MNIST 的图像分类结构
![][image-1]
## 2D + CNN
``` python
from __future__ import print_function
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()
from tensorflow.examples.tutorials.mnist import input_data
import time
# number 1 to 10 data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)


def compute_accuracy(v_xs, v_ys):
    # global prediction
    y_pre = sess.run(prediction, feed_dict={xs: v_xs, keep_prob: 1})
    correct_prediction = tf.equal(tf.argmax(y_pre, 1), tf.argmax(v_ys, 1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
    result = sess.run(accuracy, feed_dict={xs: v_xs, ys: v_ys, keep_prob: 1})
    return result

def weight_variable(shape):
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)

def bias_variable(shape):
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)

def conv2d(x, W):
    # stride [1, x_movement, y_movement, 1]
    # Must have strides[0] = strides[3] = 1
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
    # stride [1, x_movement, y_movement, 1]
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')

# define placeholder for inputs to network
xs = tf.placeholder(tf.float32, [None, 784])  # 28x28
ys = tf.placeholder(tf.float32, [None, 10])
keep_prob = tf.placeholder(tf.float32)
x_image = tf.reshape(xs,[-1,28,28,1])

## conv1 layer ##
W_conv1 = weight_variable([5,5,1,32])
b_conv1 = bias_variable([32])
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1) # x_image: 28*28*1 output size: 28*28*32
h_pool1 = max_pool_2x2(h_conv1) # output size: 14*14*32

## conv2 layer ##
W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])
h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)# 14*14*32 -> 14*14*64
h_pool2 = max_pool_2x2(h_conv2) # 14*14*64 -> 7*7*64

## func1 layer ##
W_fc1 = weight_variable([7*7*64,1024])
b_fc1 = bias_variable([1024])
h_pool2_flat = tf.reshape(h_pool2,[-1, 7*7*64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1)+b_fc1)
h_fc1_drop = tf.nn.dropout(h_fc1,keep_prob)

## func2 layer ##

W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])
prediction = tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2)+b_fc2)

# the error between prediction and real data
cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction),
                                              reduction_indices=[1]))       # loss
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

sess = tf.Session()
# important step
# tf.initialize_all_variables() no long valid from
# 2017-03-02 if using tensorflow >= 0.12
if int((tf.__version__).split('.')[1]) < 12 and int((tf.__version__).split('.')[0]) < 1:
    init = tf.initialize_all_variables()
else:
    init = tf.global_variables_initializer()
sess.run(init)
start = time.time()
for i in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    sess.run(train_step, feed_dict={
             xs: batch_xs, ys: batch_ys, keep_prob: 0.5})
    if i % 50 == 0:
        print(compute_accuracy(
            mnist.test.images[:1000], mnist.test.labels[:1000]))
end = time.time()
print('execution time: ' + str(end-start) + 's')
```
## 2D + CNN + Ray
``` python

from tensorflow.examples.tutorials.mnist import input_data
import os
import ray
import time
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()

ray.init(num_gpus=8, include_webui=True)
# consruct neural network

def weight_variable(shape):
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)

def bias_variable(shape):
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)

def conv2d(x, W):
    # stride [1, x_movement, y_movement, 1]
    # Must have strides[0] = strides[3] = 1
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
    # stride [1, x_movement, y_movement, 1]
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')

def construct_network():

    # define placeholder for inputs to network
    xs = tf.placeholder(tf.float32, [None, 784])  # 28x28
    ys = tf.placeholder(tf.float32, [None, 10])
    keep_prob = tf.placeholder(tf.float32)
    x_image = tf.reshape(xs, [-1, 28, 28, 1])

    ## conv1 layer ##
    W_conv1 = weight_variable([5, 5, 1, 32])
    b_conv1 = bias_variable([32])
    # x_image: 28*28*1 output size: 28*28*32
    h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
    h_pool1 = max_pool_2x2(h_conv1)  # output size: 14*14*32

    ## conv2 layer ##
    W_conv2 = weight_variable([5, 5, 32, 64])
    b_conv2 = bias_variable([64])
    h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) +
                         b_conv2)  # 14*14*32 -> 14*14*64
    h_pool2 = max_pool_2x2(h_conv2)  # 14*14*64 -> 7*7*64

    ## func1 layer ##
    W_fc1 = weight_variable([7*7*64, 1024])
    b_fc1 = bias_variable([1024])
    h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
    h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1)+b_fc1)
    h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

    ## func2 layer ##

    W_fc2 = weight_variable([1024, 10])
    b_fc2 = bias_variable([10])
    prediction = tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2)+b_fc2)

    # the error between prediction and real data
    cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction),
                                                  reduction_indices=[1]))       # loss
    train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
    
    correct_prediction = tf.equal(tf.argmax(prediction, 1), tf.argmax(ys, 1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

    return xs, ys, train_step, keep_prob, accuracy

@ray.remote(num_gpus=1)
class CNN_ON_RAY(object):
    def __init__(self, mnist_data):
        self.mnist = mnist_data
        # Set an environment variable to tell TensorFlow which GPUs to use. Note
        # that this must be done before the call to tf.Session.
        os.environ["CUDA_VISIBLE_DEVICES"] = ",".join(
            [str(i) for i in ray.get_gpu_ids()])
        with tf.Graph().as_default():
            with tf.device("/gpu:0"):
                self.xs, self.ys, self.train_step, self.keep_prob, self.accuracy = construct_network()
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
                          self.xs: batch_xs, self.ys: batch_ys, self.keep_prob: 0.5})

            if i % 50 == 0:
                # print(compute_accuracy(
                #     mnist.test.images[:1000], mnist.test.labels[:1000]))
                print(self.get_accuracy())

    def get_accuracy(self):
        return self.sess.run(self.accuracy, feed_dict={
            self.xs: mnist.test.images[:1000], self.ys: mnist.test.labels[:1000], self.keep_prob: 1})

# load MNIST dataset，并告诉Ray如何序列化定制类。
mnist = input_data.read_data_sets("MNIST_data", one_hot=True)
start = time.time()
# Create the actor. 实例化actor并运行构造函数
nn = CNN_ON_RAY.remote(mnist)

# Run a few steps of training and print the accuracy.
train_id = nn.train.remote(1000)
ray.get(train_id)
end = time.time()
print('execution time: ' + str(end-start) + 's')
```

## 结果分析

## No Ray
```
0.087
0.749
0.854
0.887
0.902
0.926
0.919
0.941
0.946
0.949
0.954
0.953
0.958
0.956
0.96
0.965
0.962
0.963
0.965
0.968
execution time: 52.58758211135864s
```

## Ray
```
(pid=11597) 0.077
(pid=11597) 0.722
(pid=11597) 0.853
(pid=11597) 0.877
(pid=11597) 0.904
(pid=11597) 0.912
(pid=11597) 0.921
(pid=11597) 0.932
(pid=11597) 0.94
(pid=11597) 0.938
(pid=11597) 0.944
(pid=11597) 0.941
(pid=11597) 0.944
(pid=11597) 0.952
(pid=11597) 0.953
(pid=11597) 0.951
(pid=11597) 0.957
(pid=11597) 0.959
(pid=11597) 0.958
(pid=11597) 0.966
execution time: 59.315696001052856s
```

从结果看，Ray在性能上没有进步太多，这是因为在运行过程中并没有使用多 worker，也就是没有发挥 Ray 本身（分布式框架）的属性。因为是前期实验，所以没有太多更复杂的操作，多worker并行操作就需要涉及到不同worker之间 weight, bias的同步以及网络结构的统一，这都是需要在后期考虑的事情。
![dashboard][image-2]

## 问题
通过和同学的讨论，之后会选择在终端先启动 Ray，然后在 ray.init() 中进行调用的方式进行训练，这样就避免了每一次任务结束之后 Ray 会自动关闭的情况，但在测试中发现了一些问题，进行记录。
1. 在终端中启动 Ray 之后，会进行一下输出 

	```
	Started Ray on this node. You can add additional nodes to the cluster by calling
	ray start --address='172.17.78.111:21907' --redis-password='5241590000000000'
	```

	其意思为增加新的节点，在测试过程中我一共输入了两遍，逻辑上共创建了1个母节点（master）和两个子节点（slave），但在 dashboard输出中，我并没有看到3个节点间的逻辑关系
	![ray-test1][image-3]
	![ray-test2][image-4]
2. 终端运行 `python3 cnn-ray.py` , 在 dashboard 中可以看到，的确只有一个进程（worker）在执行task。
	![ray-test3][image-5] 
	但是在终端输出界面，却发现了同样的结果输出了3次的情况
	![ray-test4][image-6]

[image-1]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/cnn-str.jpeg
[image-2]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-dashboard.png
[image-3]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-test1.png
[image-4]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-test2.png
[image-5]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-test3.png
[image-6]:	https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/ray-test4.png