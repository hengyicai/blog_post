---
layout: post
title: "Decoding in Seq2Seq"
date: 2018-05-30 11:34:11 +0800
comments: true
categories: "Machine Learning"
tags: [采样方法, seq2seq, beam search]
keywords: 采样方法, seq2seq, beam search
description: "在 Encoder-Decoder 架构中，一般使用 Beam Search 来进行解码，但是该解码方式存在诸多问题。"
---

在 `Encoder-Decoder` 架构中，一般使用 `Beam Search` 来进行解码，解码过程中，始终保持`K`个最佳结果，这里的最佳是指：
最大化 `log p(T|S)`, 其中，`S` 是源语言，`T`是目标语言。即最大化对数似然(1)：

$$ \hat{T}  = argmax_{T}{log p (T|S)} $$

但是这种解码方式有以下问题：

1. Simply conducting beam search over the above objective will tend to generate generic and safe responses that lack diversity, such as “I’m not sure”. 即，解码产生的 Tok-K 个结果不具备多样性且彼此之间较为相似。
2. RNN-based neural decoder provides a poor approximation of the conditional probability log p(T|S), and instead biases towards the target language model P(T). 即，解码产生的 Tok-K 个结果与源语言 S 的关系较小，甚至无关。

在文章[2]中，作者提出使用以下目标函数(2)来指导解码过程:

$$\hat{T} = argmax_{T}\{log p(S|T) + log p(T)\} $$

并且，作者指出，由于 $p(T|S)$ biased towards $p(T)$, 因此，实际上，上述目标函数也一定程度上包含了 `Maximum Mutual Information(MMI)` 目标函数.

接下来的问题是，如何设计一种高效的搜索算法来寻得 K 个使得目标函数(2)最大的结果。

文中作者参考 [1] 中提出的方法：generates multiple target hypotheses with stochastic sampling based on p(T|S) and then ranks them with the objective function (2) above.

作者指出，这种采样方法（stochastic sampling）仍旧存在问题，即：step-by-step naive sampling can accumulate errors as the sequence gets longer.

为了解决该问题，作者提出，使用 Selective Sampling 来进行采样，而不是直接随机采样，大体思想如下：
使用一个 sample selector，该 sample selector 是一个多层感知机，输入为：

1.  the log-probability of current sample word in $p(w_t|S)$;
2.  the entropy of current predicted word distribution, $\sum_{w_t}{P(w_t|S)log P(w_t|S)}$ for all $w_t$ in the vocabulary;
3.  the log-probability of current sample word in $p(w_t|\emptyset)$.

输出为一个二值变量，表示接受或拒绝当前采样结果。

使用 Selective Sampling 进行采样的过程中，如果N个随机采样的结果一个都没被接受，则取selector得分最高的那个；如果同时多个结果都被接受，则随机选择一个，并且，作者使用selector的得分来作为 P(T) 的估计。

在这种 selective sampling的基础下，作者首先 sampling 出若干候选者，之后，再从这些候选者中选出使得目标函数(2) 得分最高的前K个结果。

相比 naive-sampling，这种 sample-then-select 的方法既可以产生更具多样性的结果，又可以保持语言的流畅性。

这里的采样过程和统计上的采样相比，两者的目的是不同的，前者是为了让结果具有某种偏好，后者一般是为了估计总体的某种统计性质。

参考文献
[1] Tsung-Hsien Wen, Milica Gasic, Nikola Mrksic, Peihao Su, David Vandyke, and Steve J. Young. 2015. Semantically Conditioned LSTM-based Natural Language Generation for Spoken Dialogue Systems. Proceedings of the 2015 Conference on Empirical Methods in Natural Language Processing pages 1711–1721.

[2] Di Wang, Nebojsa Jojic, Chris Brockett, and Eric Nyberg. 2017. Steering output style and topic in neural response generation. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing, EMNLP 2017, Copenhagen, Denmark, September 9-11, 2017, pages 2140–2150. Association for Computational Linguistics.

