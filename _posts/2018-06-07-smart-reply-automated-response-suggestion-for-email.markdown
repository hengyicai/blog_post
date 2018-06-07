---
layout: post
title: "Smart Reply: Automated Response Suggestion for Email 阅读笔记"
date: 2018-06-07 15:26:11 +0800
comments: true
categories: "Dialogue System"
tags: [对话生成, Deep Learning]
keywords: 对话生成, Deep Learning
description: "本文来自 Google，发表于 KDD 2016, 文章提出了一种用于 Gmail Inbox 中对邮件快速回复的系统."
---

本文来自 Google，文章提出了一种用于 Gmail Inbox 中对邮件快速回复的系统，系统整体流程如下：

![](https://ws2.sinaimg.cn/large/006tKfTcly1fs2ne0s8bwj30ey0iowge.jpg)

系统 Demo 如下：

![](https://ws2.sinaimg.cn/large/006tKfTcly1fs2neyoxn0j309r0hd40o.jpg)

本文的方法属于检索式的方法，训练的 Seq2Seq 是用来做 Scoring，并不是用来做生成的。精心构造好候选集之后，使用 LSTM 做 Ranking。
从该系统中可以看出，工业界可用的线上系统一般需要多方面细致的考虑。在整个流程中，作者加入了多种机制来达到如下目标：

1. Response quality. 由于是线上产品，因此系统必须给出高质量的回复。作者特别指出：“ensure that the individual response options are always high quality in language and content.”.
2. Utility. 系统必须 “select multiple options to show a user so as to maximize the likelihood that one is chosen.”，即，回复要具有多样性。使得这些回复尽可能地被用户使用（选中其中某一条）。
3. Scalability. 系统必须能够每天处理百万级别的消息且始终做到低延迟。
4. Privacy. 对隐私的保护。这点作者在文中只是做了说明。文中主要对前三点要求做了方法上的陈述。

系统主要的模块如下：

1. Response selection: 使用 LSTM 来对候选集进行 Ranking. 
2. Response set generation: 候选集的生成。作者使用半监督的图学习方法来生成候选集，该过程是离线的。
3. Diversity: 作者采用多种机制来提高回复的多样性。
4. Triggering model: 该模块作为一个二分类器，来决定是否要对某封邮件作出 reply，作者在实验部分指出，只有大概11%的邮件需要给出自动回复建议，其他的则不需要（自动生成的邮件，无针对性的通知性或开放性的邮件等）。作者指出，尽可能地减少不必要的回复带来的好处是算力的低开销。

下面简单对各个模块做下介绍：

## Response selection:
这个模块主要使用了一个 Seq2Seq 的模型来计算某个 candidate sentence 的概率值：

![](https://ws1.sinaimg.cn/large/006tKfTcly1fs2ngcgasfj30ct020aa2.jpg)

这个很简单，就是在 decoding 的过程中，累乘目标 token 的概率值即可。模型的训练则是用 $(message, response)$ 对儿来进行训练。

## Response set generation:
该模块的目的就是产生一个 response 集合，该集合尽可能覆盖较多的回复类型，来保证多样性。作者在这里使用图聚类的方式来产生多个 cluster，每个cluster对应与某一种 intent，cluster 内包含有多条相同 intent 的 response。 比如 “Ha ha”，“lol”， “Oh that’s funny! ” 同属于 funny 这个 intent。
图是怎么构造的呢？ 图中有两类节点，response 节点和 feature 节点，response 中的 ngrams 和 skip-grams 作为 该 response 的 features。
如果 节点 feature 属于 response，那么这两个顶点之间有边连接。这样一来，具有相似 feature 的 response 节点也会具有较近的距离。
作者首先人工标记了若干 response 节点的 intent label (大概三五百个)，然后使用 EXPANDER 算法对图进行聚类，使得 intent 传播至其他未标记的 response 节点。具体过程就是最小化如下两个损失函数（分别对应于 response 节点 和 feature 节点）：

![](https://ws4.sinaimg.cn/large/006tKfTcly1fs2nhcfiu1j30ch0ctwhs.jpg)

该模块的示意图如下：

![](https://ws1.sinaimg.cn/large/006tKfTcly1fs2nfv0isdj30mr0b6gob.jpg)

这里有一个问题，作者提供了若干种子节点（有label的）来对其他 response 节点进行 label 传播（通过聚类），那么，如何发现新的 label(intent) 呢? 
可能某些 response 本身不属于任何一个 intent。对于这个问题，作者的方法如下：

首先执行聚类算法5次，那么现在图中可能仍有未被 label 的 response 节点，然后随机从这些 未label 的 response 节点中 sample 出 100 个出来，
并且分别以各自的 canonicalized representation 作为各自的 label，然后重复这个过程直到收敛（所有的 response 节点均被 label 且 类内成员不再发生变化）。
这里的  canonicalized representation 其实就是 该 response 的最精简语法结构，比如说 “how_about_time” .

文章中指出， 这些 (response, cluster label) pairs 会再经过人工检查，检查之后的作为候选集来使用。

## Diversity：
这个模块分为两个步骤：

1.  Omitting Redundant Responses：对候选集里的句子逐一使用 Seq2Seq 计算得分之后，排名取前K 个，然后去掉 intent 重复的候选句子( 优先去掉得分低的 )；
2.  Enforcing Negatives and Positives：如果 Tok-K 个都是 Positive 的回答，则将候选集中的 Negative response 送入 Seq2Seq 中计算得分，得到一个 negative 的最优 response，插入到 结果中（结果一般取三个response）。反过来，如果一开始得到的都是 Negative 的，则执行上述过程以得到一个 Positive 的，最终目的是为了提高 Diversity。

## Triggering model：
这个模块其实是一个 MLP 二分类器，数据形式为 $(message, True/False)$, 表示该 message 是否得到了回复。

对于作者提出的这个系统，文章最后结合其他 baseline 做了准确率，MRR(Mean Reciprocal Rank) 方面的对比，并针对系统的使用情况给出了一些量化结果，比如：使用推荐 reply 的用户占比大概10%，在所有使用推荐 reply 的用户中，45% 的用户选择了第一个, 35% 的用户选择了第二个，20% 的用户选择了第三个。

从这篇工作来看，有三点思考：

1. 工程上的打磨对工业界的产品来说更加重要。文中使用的模型和算法并不复杂，但是整个系统结合多个模块针对各种实际问题作出了较好的解决。
2. 目前，完全 Data-Driven 的生成式方法在结果质量, 结果多样性，个性化上仍需继续探索。



