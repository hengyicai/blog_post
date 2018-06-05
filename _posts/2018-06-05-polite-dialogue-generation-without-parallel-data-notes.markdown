---
layout: post
title: "风格化对话生成--论文阅读笔记"
date: 2018-06-05 17:30:11 +0800
comments: true
categories: "Dialogue System"
tags: [对话生成, 风格化, Deep Learning]
keywords: 对话生成, 风格化
description: "作者提出三种模型来完成风格化对话生成的任务"
---

文章来自: http://arxiv.org/abs/1805.03162, 出发点是风格化对话的生成.
 
考虑到风格化对话 <regular-to-stylistic> pairs unavailable, 作者提出三种模型, 使用弱监督的方式, 

* 结合通常的 encoder-decoder 模型 + pre-trained language model.
* label-fine-tuning(LFT) model.
* Polite-RL, encourages politeness generation by assigning rewards proportional to the politeness classifier score of the sampled response. 

Each of our three models is based on a state-of-the-art politeness classifier and a sequence-to-sequence dialogue model. 关键的两个组件是 `Seq2Seq` 和 `style classifier`.

作者 also present two retrieval-based polite dialogue model baselines. 基于检索的baseline比较简单, 此处略去了, 基本就是对candidates中的response做matching,然后取得分最高的作为 response.

作者指出问题的挑战性:

Generating stylistic dialogue responses is a substantially challenging task because the generated response needs to be syntactically and semantically fluent, contextually-relevant to the conversation, as well as convey accurate paralinguistic features. 

作者指出, 这个问题在缺乏平行语料的情况下尤为突出.

共三个模型: Fusion, LFT, Polite-RL.

先介绍下相关工作:

## 相关工作

### Models for Style Transfer

#### Style Transfer with parallel Data

作为翻译问题, 将源语言翻译为具有某种风格的语言.
提到的方法: labeled sequence transduction methods, 向 model 中加入某种 label, 该label可以由多种方式获得, 比如本文中使用 style classifier 来获得.

#### Style Transfer without Parallel Data

人工在目标语言中进行标记引入 label , 或者, 采用自编码器, cross-alignment 来分离 style 和 content, 类似于李飞飞组做图像的风格化迁移时的做法.

但是作者指出, 本文工作和 style transfer 最大的不同在于, style transfer 需要 **“strict content preservation”**, 即需要保持内容一致, 在这个前提下, 引入 style.
而本文是在对话场景下, our task is about maintaining good relevance to the context when adding style, especially useful for dialogue-based tasks.

### Multi-Task Learning and Style Transfer

(Luan et al, 2017) 提出一种 MTL 的方法来引入风格, 首先 train 一个 Seq-to-Seq 的 model(使用平行语料), 然后使用自编码器来训练 persona-related data from target speakers, 之后, share the decoder parameters of these two models so that the generated responses can be adapted to the style of the target-speaker. (参数共享的思想)

本文作者提出的 Fusion 模型是在 inference 阶段来  combine the parameters of the language model(the polite style) with a generative dialogue model trained on parallel data.

## 三种模型

首先是 politeness 分类器:

![](https://ws1.sinaimg.cn/large/006tKfTcly1fs0gkj0532j30dr0g3gn4.jpg)

### Fusion Model

![](https://ws3.sinaimg.cn/large/006tKfTcly1fs0gpu5h9uj30f909a75a.jpg)

首先, 使用上面的 classifier 来过滤出一部分 “polite” 得分较高的 句子, 然后, 使用这些句子来训练一个 polite-LM (2-layer LSTM-RNN), 
然后在 Seq-to-Seq 解码的时候, 在每一个 timestep, 都得出一个 fused distribution:

![](https://ws1.sinaimg.cn/large/006tKfTcly1fs0gn38y6zj30fd08hgn0.jpg)

然后, 在每一步得出 prediction 的时候, 用这个 $p_t$ 来得出结果.

### Label-Fine-Tuning Model:

很显然, Fusion model 有两个显著缺点:

1. output 中很大一部分是直接由 “polite language model”来决定的, 因此, 产生的 output 没有结合对话场景中上下文 context 的信息,
容易产生不相关的response.
2. Fusion model 只是在 inference 的时候加入 polite 信息, 并没有 “learn politeness during training”.

为了解决以上两个问题, 作者提出 “prepends a predicted continuous style label at the beginning of each input sentence to specify the intended politeness level.”, 
模型如下:

![](https://ws4.sinaimg.cn/large/006tKfTcgy1fs0gs38mdlj30fw0d5jt0.jpg)

在训练的时候, 通过向词表中加入一个特殊的label, 该label的embedding也是可训练的, 使用 target 的分类器得分值来对其进行 scale, 使得: 
the label and the source sentence cooperatively determine what the overall response looks like.

while during test time, we are free to scale the prepended politeness label with different scores of our choice (i.e., when we want the model to generate a polite response, we scale the label’s embedding by a score between 0.5 and 1.0, while to generate a rude response, we scale the embedding by a score between 0.0 and 0.5).

### Polite Reinforcement Learning Model:

向模型中新增 $Loss_RL$ , 来引入 polite 信息, 两项 Loss 分别为:

![](https://ws3.sinaimg.cn/large/006tKfTcly1fs0gvmqy79j30ec02w0ss.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcly1fs0gvtfmxxj30er03faa6.jpg)

模型如下:

![](https://ws2.sinaimg.cn/large/006tKfTcly1fs0gxjwhclj30w10c776q.jpg)

## 实现上的细节:

* Embedding Initialization. word2vec trained on Google News dataset.
* Pre-training, 首先使用平行语料做预训练.
* 优化器使用 Adam, 学习率为 0.001, dropout 0.2, mini-batch size 为 96

## Conclusion

作者在结论部分指出， one could employ more recent, advanced models such as variational, adversarial, and decoder-regulation techniques.

## 个人思考:

本文更多的是在 style 上来做, 这里的 style 是“通用的”, “共有的”, 不属于 personality, 本文工作更像是将 “style-transfer” 这个任务放到了对话场景中(引入的要求就是 coherent with context, 而不是像 style transfer 中的 content preservation ).



参考文献
[1] Yi Luan, Chris Brockett, Bill Dolan, Jianfeng Gao, and Michel Galley. 2017. Multi-task learning for speakerrole adaptation in neural conversation models. In Proceedings of IJCNLP.

