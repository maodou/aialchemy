---
title: Why We Need More Compute for Inference
summary: 
description: 
categories: 致敬MP
tags: 
externalUrl: 
date: 2024-05-03T07:25:14+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-05-11T02:24:05+08:00
share: true
---
[Why We Need More Compute for Inference](https://www.deeplearning.ai/the-batch/why-we-need-more-compute-for-inference/), Andrew Ng, Andrew' Letters

> Much has been said about many companies’ desire for more compute (as well as data) to train larger foundation models. I think it’s under-appreciated that we have nowhere near enough compute available for inference on foundation models as well.

开篇明义，计算资源不仅依旧无法满足现有大模型训练的需求，大模型对外输出的推理服务所需的计算资源也是捉襟见肘，完全被低估了。

计算资源方面的疑虑，在 AI 爆发前期就已经存在。疑虑来自于个人通用计算资源的类似对比，对于大部分普通用户而言，CPU 等计算资源早已远远过剩，部分专业人士、游戏玩家除外。

接下来具体讲解了为什么现有的 AI 计算资源还远远不够无法满足 AI 推理服务的需求。

>Today, a lot of LLM output is primarily for human consumption. A human might read around 250 words per minute, which is around 6 tokens per second (250 words/min / (0.75 words/token) / (60 secs/min)). So it might initially seem like there’s little value to generating tokens much faster than this.

另一个对计算资源过剩的考量来自于 LLM 的使用现状。目前，LLM 的主要消费群体还是人类。并给出了如下数据作为旁证：人类的阅读速度在 6 token/秒左右，对计算资源的要求似乎并不苛刻，看起来现有的计算资源能够支撑。

但人类直接消费的场景只是 AI 应用很小一部分，如果考虑到 RAG 等其他领域的 AI 应用，这些计算资源是远远不够的。 

> Fortunately, it appears that both training and inference are rapidly becoming cheaper.

最后，给了一个令人兴奋的预测，随着大语言模型算法和硬件能力的提升，训练同等量级的大模型成本能够下降 75%，推理的成本更是能达到降低 86% 水平。这里他点名了一家公司 [Groq](https://groq.com/)，一家加速大模型推理服务的公司，以及另一个 [SambaNova](https://fast.snova.ai/) Demo。

算法、硬件的提升让训练和推理成本急剧下降，并不意味着对大语言模型的计算资源的需求量少了。AI 的能力远未见顶，计算资源和数据对 Transformer 架构的大语言模型的提升还有很大的提升，Llama 3 就是很好的例证。