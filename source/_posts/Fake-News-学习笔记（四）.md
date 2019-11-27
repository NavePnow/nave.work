---
title: Fake News 学习笔记（四）Project-Summary NLP-Summary
tags: [NLP, Python]
categories:
  - FakeNews
toc: false
date: 2019-08-16 18:03:57
thumbnail: https://images.unsplash.com/photo-1434030216411-0b793f4b4173?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60
---

转载大佬的博客，写的太牛逼了: [从Word Embedding到Bert模型—自然语言处理中的预训练技术发展史]
<!--more-->
## 牢骚
为期6周的项目终于结束了，从昨天晚上连夜训练跑代码到今天的 Presentation，虽然还有点紧张，但更多的是对项目的无奈，一组4 个人，连带 Q&A 共 15min，平均下来每个人只有 2 分半的时间去展示自己做的东西。我实话说吧，就光 Bert，我都可以讲5min，一共 9 个组，难道就不能分来？？哪怕 3 个 3 个来也行，说起来我是真的烦，既然就是收钱那推荐信的项目了，掏了几万块钱就不能让这个钱花的值一点？其实我在小组分工方面是有点偏心的，我是想做文字处理的部分，而且最后选择了 Bert ，因为我想尝试一些不一样的东西，分类器之前大三下有上过课，虽然学的不是特别懂，但总算是有接触过，大学也不就是通识教育吗，我也没有希望我能有多么深入的学习，只希望我可以多学一点不会的东西，对于 NLP，我之前是一点都没有接触，当然原理性的东西就根本不可能知道，得知了项目是做关于自然语言处理的，我有蛮感兴趣，通过课上的学习，教授说了很多新兴的自然语言处理工具，其中就包括了 Bert，Bert 是 18 年底提出的知识，总感觉是我在大学学习的最新的知识了吧，相比于 11 年的 C++课本。。。不管了，知识是学给自己的，谁也偷不走，最后我还是总结一下我总的学到的知识吧。下面英文的部分是 “官方要求的项目总结”，就不再多说了，关于 Bert 的实现在另外一篇博客中有所展示，这里就不再一一赘述了。

## 个人理解 从 Word2Vec 到 Bert
从最开始的 Word2Vec，到 GloVe，一直到最新的 Bert，无非是将文字转化为一个可以让机器识别的数据格式-矩阵。在最开始的模型中，也就是 Word 和 Glove，存在一个初始的矩阵，里面包含了该方法所用的文字 token 以及对应的 value，只需要将文字输入到模型中，即可得到每个文字所对应的向量值，也就是矩阵值(相乘)，这样就涉及到了如何得到初始矩阵的问题，Word 和 Glove 使用的方法当然是不同的，在我们项目小组中，我们尝试了用 Glove 代替 Word，并取得了较好的效果，但实质还是没有变化，存在的问题也是显然的。我们都知道每个单词在句子中的意思是依附于上下文的，就比如所有博客中提到的 Bank 这个单词，有银行和河岸的意思，所以将所有的意思用一个矩阵去表示显得有失偏颇，所以基于两者提出了更多的模型。

这就要提到 Elmo了（我学的不是很深，只是把大概的思想了解了一下）：根据上下文关系进行向量的生成，但是有别于前面两者，什么意思。Word 和 Glove 只是把初始矩阵给你，你并不可以改变其中的值，但是在 Elmo 中，应该是使用了迁移学习的思想，在初始矩阵的基础上根据训练集进行相关的更新，也就是矩阵值的更新，但是怎么生成初始矩阵我并没有怎么学习，Bert 倒是了解了一点，反正对于我来说，Elmo 的意义就是矩阵的动态更新，使得矩阵里面的值可以更贴切训练集中特定的文本。而且从训练效果看，具有很大的提高，说到这里，其实有点鄙视最开始的 Word 和 Glove 了，因为考虑的东西太少，这也是为什么我看到别人用 Word2Vec 可以达到0.9多正确率而直呼牛逼的原因。

从 Elmo 到 Bert，又是一个飞跃，当然不能否认，Bert 是基于前人的工作而提出的新型的模型，因为很多思想和前者都有很多相似之处，可以说是站在巨人的肩膀上看世界。Bert 采用了更为高效的网络结构-Transformer来更新初始矩阵, 以及巨大的预训练模型用来生成初始矩阵。同样是考虑到了文章的上下文关系，在 Bert 中共有 3 个不同的变量作为模型的输入，每个单词的 Embedding，位置的 Embedding 和句子的 Embedding（可以理解为和初始矩阵的乘积？），这些都是依附于初始矩阵得到的结果，但是有文章说 Word 的 Embedding 不依附于初始矩阵，那我就不是很懂了，anyway，有三种不同的输入进行网络模型。
在模型中只使用到了 Transformer 的 Encode 部分，具体的工作流程我就不懂了，反正经过了多层的 Encode，伴随着矩阵的更新（fune-tuning），最终可以得到很多个向量的结果，再加上 linear 和 softmax 层即可解决分类问题。有人会问了，那 Bert 是如何得到初始矩阵的。这就是 Bert 牛逼之处了，通过 mask 遮盖句子中的单词来训练单个单词，通过遮盖整个句子来训练整个句子，听过 Google 训练了整个 Wikipedia，真的牛逼。最终通过矩阵的更新得到了相应的结果。利用 Transformer 加上强大的与训练模型使得 Bert 脱颖而出，摘得了 NLP 的头牌，有人说 Bert 是 NLP 的顶峰，我看了几篇文章，一个是清华大学的，一个是斯坦福大学的，都是基于 Bert 的现有模型，要么是修改网络结构，要么是增加模型的输入以提高文章的 Contextual 属性，也得到了比初始模型更好的结果，可以说是炼丹成功的典范了。

## 总结
说到这里，项目也就结束了，从最开始的打开 FakeNewsTutorial 到写下这最后一篇博客，完成了项目所有的工作，还8.23 下午赶了 Github 的 repo 作为展示，修改中介关于项目的细节，以及项目的总结书，感谢组员的配合和合作，我感觉我们在最后的 Pre 发挥的很好，希望给教授留下不错的印象吧，希望大家都好好努力，前程似锦。
																——王依凡 08/24/2019 华科东 11 栋



## Project Summary
### 1. Contribution
 1. My job in our group is to use Google Bert to classify the text and complete a pipeline.
 2. Use various word processing methods for specific text to improve the final f1 score (0.986)
 3. I arrange member’s work within the group and meeting time. Three rehearsals were completed through communication with members before the final pre.
 4. Use my own server to build a python environment for team members
 5. Check out the relevant information and code about Glove on the Internet and use it as a demo for other members.

### 2. Idea
 1. Specific text optimization for specific texts, such as fake_or_real_news.csv. Because Bert learns each sentence and the relationship between the sentences, using regular expressions to process non-sentences in the article (such as #hashtag) is necessary.
 2. By checking out the relevant information, I learned that EDA (Easiest Data Augmentation) can improve the classification in the case of small text, so I tried two methods of EDA, one is Random Insertion, the other is Synonym Replacement)
 3. Use Google Bert as a method of text classification. Google Bert is a cutting-edge approach to do NLP tasks with excellent and efficient classification and I use the Bert model with the softmax layer to complete text classification.

### 3. Something I want to share
The initial idea of our group was to use Google Bert to generate vectors for downstream text classification tasks. However, through code testing, the output includes word vectors and sentence vectors with high dimensions and huge time consumption. Therefore, extracting vectors separately from the Bert model is a bit hard. If the vector can be extracted, the output also includes the word vector and the sentence vector, which is difficult to handle. Thus, we used the method mentioned in the article- adding softmax layer to finish the text. Then, I begin my work to use a completed Bert pipeline to finish the job. 

At the beginning, I spent a lot of time collecting various information about Bert. Because Bert is based on the existing NLP model, I started with the history of NLP and learned the principles of multiple models, from GloVe to LSTM and ELMo, and finally to Bert. I finally got a glimpse of how Bert works and wrote a blog to my URL: https://nave.work . 

In fact, I used the original data to feed the model and got a 0.98 f1 score, so how to optimize on this basis becomes a problem. I observed a lot of content that was not a sentence by observing the text, including the URL and #hashtag, so I used regular expressions to remove specific content to improve the f1 score, and finally reached the score of 0.986.

In the final training process, due to the tight time and heavy tasks, I rented 1080ti to complete all the tasks. Of course, I am satisfied with the final result. This is based on the powerful per-training model of Google Bert. Anyway, I gained a lot of knowledge through this project and learned a lot of debugging skills, I love NLP!! (Finally attach my Contribution chart: 
![](https://i.loli.net/2019/08/23/1dpgIJrTla3Uq6w.png)
and Github repo: https://github.com/NavePnow/Google-BERT-on-fake_or_real-news-dataset#5-part4-reference