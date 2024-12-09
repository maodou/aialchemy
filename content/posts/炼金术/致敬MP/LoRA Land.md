---
title: LoRA Land
summary: 
description: 
categories: 致敬MP
tags: 
externalUrl: 
date: 2024-05-06T22:20:52+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-05-11T02:24:05+08:00
share: true
---
[LoRA Land: 310 Fine-tuned LLMs that Rival GPT-4, A Technical Report](https://arxiv.org/abs/2405.00732), Justin Zhao ..., [Predibase](https://predibase.com/)

### Abstract
> First, we measure the quality of LLMs fine-tuned with quantized low rank adapters across 10 base models and 31 tasks for a total of 310 models. We find that 4-bit LoRA fine-tuned models outperform base models by 34 points and GPT-4 by 10 points on average. Second, we investigate the most effective base models for fine-tuning and assess the correlative and predictive capacities of task complexity heuristics in forecasting the outcomes of fine-tuning. Finally, we evaluate the latency and concurrency capabilities of LoRAX.

摘要明确了这篇论文的三个主要成果，
1. 评估 310 个 4 bit LoRA 微调模型后，发现微调后的 LoRA 模型在性能上要普遍优于基础模型和 ChatGPT4，分别高 34% 和 10%。不同基础模型以及对应的 LoRA 微调模型性能比较如下图。

	![Average model performance for GPT-3.5, GPT-4, and 310 LLMs, before and after fine-tuning with LoRA, across 31 different tasks and 10 different base models](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240506223312.png)

	310 个模型由 10 个基础模型，分别在 31 个任务数据集上 LoRA 微调而来，10 * 31 = 310。
	
	如果没有理解偏差，这是车轮战的节奏。每个基础模型上分别训练 31 个不同任务数据集的微调模型，然后在对应的 31 测试任务数据集上进行评估，最终得出上述 34% 和 10% 的结论。第一感觉，比很多“泄题考”的模型还不。。。。。。虽然在论文的后半部分提到，在模型评估上如何比其他模型公平（主要指 prompt 设计方面）。
	
	无论怎样，基于论文的前提假设 -- LLM 的未来是垂直领域模型，如此评估比较倒也无可厚非。虽然论文中没有提及，其他类似新模型发布的论文中也不会提及，但是仍然需要澄清的一点是：这个结论知识基于这些已经被用烂了的评估数据集上的结论，而不是实际使用效果。

	LLM 后续发展无法确保，但是从现有模型来看，7B 左右的模型，包括最近发布的 llama3-8B 无论如何微调都无法和 ChatGPT4 相提并论。这个量级的基于 Transformer 架构的模型，及时在评估集上得分不错，但实际使用过程中幻觉、不稳定性等等，真的是谁用谁知道！

2. 挖掘最好的基础模型，并用一种启发式的方法来分析和预测微调模型的性能。论文第 5 节有相关描述，但我没看出来哪方面做到了“启发式”？可能大家都爱用这个词吧。

3. 使用了经济实惠的 LoRAX 框架来部署微调模型。LoRAX 在上篇[LoRAX -- 以一当百](https://mp.weixin.qq.com/s/xri9AGDyt-8ryTnyfm1w9A)中有详细介绍。

### 1 Introduction

Introduction 总起全文，并引出了以下几个前提假设，1）模型微调的好处及 LoRA 微调方法；2）现有的开源基础模型需要进一步微调来增强模型性能；3）Prompt 涉及影响模型评估结果，为后续论文中设计的简单 prompt 体现公平性。

最终推倒出论文的主要工作：比较分析 310 个微调模型，并采用 LoRAX 框架来更有效的承载相关实验。

### 2 Related work
论文中涉及 LLM 技术相关的工作，主要与 PEFT 模型微调相关，采用的也是主流的 LoRA 方法，并使用 4-bit 量化。

> Parameter-Efficient Fine-Tuning (PEFT) methods are designed to reduce the high expense of fine-tuning large-scale models. They achieve this by training a relatively small sub-set of parameters, compared to the total number of parameters, for adapting to downstream tasks. Existing PEFT strategies can be divided into two categories: Prompt-based methods add extra soft tokens (prompts) to the initial input and focus solely on fine-tuning these trainable vectors. Adapter-based methods introduce additional trainable modules into the original frozen backbone.

文中精炼地概括了 PEFT 的概念，以及两种实现 PEFT 的策略，1）基于 prompt 的方法，清华大学智谱清言的 p-tuning 就属于这类方法；2）基于适配器的方法，现在主流的 LoRA 是这类方法的典范。

### 3 Methodology
论文采用的方法包括以下 5 方面：
#### 任务（评估数据集）类型的选择

评估数据集来自 HuggingFace 和 Kaggle 两个主流 AI 平台，并采用公认的，不同类型的数据集。以确保评估数据集的普适性、多样性、可获取性。

#### Prompt 的选择
> Previous studies have demonstrated the potential of leveraging prompt engineering techniques, such as the use of majority voting, the inclusion of multiple in-context examples (n-shot) , MedPrompt , chain-of-thought prompting, etc., to enhance model performance on specific tasks.

特殊的 prompt 设计会左右测试结果，从而影响评估的公平性。谷歌的 Gemma 就是使用了这个“技巧”（当然还有其他技巧，比如泄题），以获取好看的评测分数。

> Instead, we opt to use simple zero or single-shot completion-style prompts for all tasks.

> we follow prescribed prompt tagging conventions for each model, as outlined in the respective model’s documentation on HuggingFace, to ensure proper querying of pre-trained and instruction-tuned base models.

因此，除了不同模型各自需要不同的特殊 token 外，其他统一采用 zero 或者 single-shot completion-style prompt。

采用 completion-style prompt 应该也是为了需要照顾评测中的 complete 模型。

#### 基础模型

基础模型的选择如下图所示，另外选用了 ChatGPT4 和 ChatGPT3.5 作为标杆模型作为对照。并且只需要一张 A10 24G 就可以进行所有模型的微调训练。
![基础模型](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240507141548.png)

#### 训练参数

进行微调训练的参数配置如下：
> Each model is trained for 40000 training steps with batch size 1, 4-bit quantization using bitsandbytes and a LoRA rank of 8. We use the paged adam optimizer, a learning rate of 0.002, and a cosine learning rate scheduler with a 0.03 warm-up fraction (1200 training steps). Gradients are applied over 16 accumulation steps for an effective batch size of 16.

采用 [Ludwig](https://ludwig.ai/)作为模型微调的框架平台。

#### 模型评估

> Non-fine-tuned models often generate more varied outputs, including unintended artifacts such as additional words or explanations not specified in the prompt.

微调之前的基础模型往往不按 prompt 出牌，比如你让它返回 0/1，它给你返回 Yes/No。为了规避这类问题，采用正则手段来匹配输出结果，从而减轻基础模型这方面的弱势。

有些测试集无法通过简单的模型输出，来做判断评估。如果上面的例子是选择题、对错题，容易机器判断。那么像 WikiSQL 等数据集就是主观题，现行的方案是通过 ChatGPT4 等公认的 LLM 来做评判。这就要涉及到经费等现实问题。

### 4 Results
![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240507144227.png)
微调模型与基础模型以及 ChatGPT4 性能比较结果如上图。再次强调一下，这只是在评估数据集上的结果。

### 5 Discussion and Analysis

分析讨论部分又回到开篇提到的 2 、 3 两个目标成果。

第2个成果是找到最好的基础模型，以及微调结果与基础模型相关性的分析和预测。

1. Mistral-7B 和 Zephyr-7b-beta 是最佳基础模型，Gemma 最差。
2. 7B 模型普遍优于 2B 模型，意料之中。
3. 指令模型（Instruction-tuned) 和补全模型（Auto-complete），微调前指令模型要优于补全模型，微调后两者不相上下。
4. ChatGPT4 在更宽泛和复杂领域具有更大的优势。

后续的启发式分析，也配备相应的统计分析，个人没看出有什么特别的价值。

最后一个目标是 LoRAX 产品介绍，就是论文第一作者所在公司的产品，[LoRAX -- 以一当百](https://mp.weixin.qq.com/s/xri9AGDyt-8ryTnyfm1w9A)中已有论述。

### 总结

论文在基于开源“小模型”微调成为垂直领域模型方向的探索，值得大家关注。LoRA 微调，以及LoRAX 多微调模型部署等方面的实践，也可以借鉴学习。但是在具体的性能提升数据，特别是优于 ChatGPT4 上，就不要太当真。