---
title: ORPO
summary: 
description: 
categories: 微调部分
tags: 
externalUrl: 
date: 2024-05-12T21:03:09+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-08-22T22:44:52+08:00
share: true
---
[ORPO: Monolithic Preference Optimization without Reference Model](https://arxiv.org/abs/2403.07691), Jiwoo Hong ..., KAIST AI

### Abstract
### 1 Introduction

首先来回顾一下模型训练三阶段：
1. Pretraining，模型在海量文本数据下进行预训练，从而具备根据输入预测输出的能力。
2. 指令微调，SFT 使模型具备某种特定能力，基本就是 Chat 模式。
3. 偏好对齐，为了解决模型产生有害的、不道德的输出的问题。RLHF、DPO 等算法就用于此阶段。

> our approach requires neither an SFT warm-up stage nor a reference model, enabling resource-efficient development of preference-based aligned models.

论文发明的 ORPO 无需复杂的微调算法（reference model）和训练方法（warm-up stage），简单且高效。下图是新方法 ORPO 与 RLHF / DPO 对比的实验结果，选用的也是公认的优秀开源模型。

![1080159178190d483505656c900e255f_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/1080159178190d483505656c900e255f_MD5.png)

### 2 Related Works

相关工作部分，回顾了早前普遍采用的对齐微调方法，RLHF 和 DPO。

#### Alignment with Reinforcement Learning

RLHF 的基本原理，使用一个增强学习算法（比如 PPO），通过人类反馈结果，最大化奖励函数，来训练模型。

也有使用语言模型代替人类进行反馈的方法，比如 RLAIF。

但这类 RLHF 方法有以下几个共同的问题，1）增强学习算法不够稳定；2）奖励函数模型也不够稳定。

#### Alignment without Reward Model

DPO 的出现就是为了缓解上述两个问题的。 

DPO 方法本身也在发展。例如，IPO 可以防止 DPO 中的过拟合； KTO 和 ULMA 在数据集方面做了优化，无需成对的偏好数据等。

#### Alignment with Supervised Fine-tuning

首先，已经有文章论证了，无论是有还是没有增强学习的情况下， SFT 在偏好对齐微调中都扮演着重要角色。

其次，SFT 微调使用的数据不在于多，而在于精。很多论文表明，挑选出部分高质量的 SFT
 数据，比使用全部数据进行模型微调的效果更好。

进一步， [Self-alignment with instruction back- translation](https://arxiv.org/pdf/2308.06259)通过模型自学习的方式进行模型训练，而且效果很不错。基本思路如下：
1. 选取少量高质量的种子数据来微调基础模型。
2. 利用微调后的基础模型，为还未标注的数据打上标注，并对标注结果做评分。
3. 选取第 2 步中评分高的数据，对 1 中的模型进行微调。
4. 重复 2 、 3 步骤两次。

以下是[Self-alignment with instruction back- translation](https://arxiv.org/pdf/2308.06259)论文中，微调 Llama-7B 的结果。选取了不同的数据集大小，和不同的评分（总分 5 分，分别选取了全部、 >4 和 > 4.5）。充分体现了数据质、量在模型微调过程中的作用，以及训练数据集选择宁缺毋滥的特性。
![59087f6465ae4731b48f10f40856ba1e_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/59087f6465ae4731b48f10f40856ba1e_MD5.png)

这种模型自学习的方法，是不是有种似曾相识的感觉？计算机语言编译器就有类似的自举机制。以 C 语言为例，
1. C 语言编译器的第一个版本通常是使用汇编等其他更底层的语言编写，实现了 C 语言的基础语法。
2. 使用 1 中 C 语言编译器版本，使用该部分 C 语言语法进一步扩充编译器的功能。最终达到现有 C 语言的能力。

### 3 The Role of Supervised Fine-tuning

不少论文表明 SFT 对于预训练后的模型，在偏好对齐、目标领域的能力增强，发挥着至关重要的作用。但是，由于 SFT 只关注解决目标结果的生成（预测概率较大的下一个 token），而缺乏对该拒绝结果（举个例子，比如让它回答“如何诈骗”，“如实回答”就是该拒绝的结果）的学习能力。因此最终的微调效果有时并不是太好。

#### Absence of Penalty in Cross-Entropy Loss

LLM 训练中普遍采用的交叉熵损失函数，是造成模型输出不想要结果的原因之一。

[一文搞懂熵(Entropy),交叉熵(Cross-Entropy)](https://zhuanlan.zhihu.com/p/149186719) 深入浅出的介绍了交叉熵以及交叉熵损失函数。便于大家理解，以下照搬了文章介绍交叉熵损失函数的例子：

---
交叉熵公式：$H(P, Q) = E_{x-P}(-log(x))$

假设一个动物照片的数据集中有5种动物，且每张照片中只有一只动物，每张照片的标签都是one-hot编码。

![773cf43fb0ffbac577bd597cb924883b_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/773cf43fb0ffbac577bd597cb924883b_MD5.png)

第一张照片是狗的概率为100%，是其他的动物的概率是0；第二张照片是狐狸的概率是100%，是其他动物的概率是0，其余照片同理；因此可以计算下，每张照片的熵都为0。换句话说，以one-hot编码作为标签的每张照片都有100%的确定度，不像别的描述概率的方式：狗的概率为90%，猫的概率为10%。

假设有两个机器学习模型对第一张照片分别作出了预测：Q1和Q2,而第一张照片的真实标签为\[1,0,0,0,0\]。

![80e00e6ee1823083a3c5f8d9269445b8_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/80e00e6ee1823083a3c5f8d9269445b8_MD5.png)

两个模型预测效果如何呢，可以分别计算下交叉熵：

![e7c3eeb9570a4b6ed6bc7410abb46805_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/e7c3eeb9570a4b6ed6bc7410abb46805_MD5.png)

交叉熵对比了模型的预测结果和数据的真实标签，随着预测越来越准确，交叉熵的值越来越小，如果预测完全正确，交叉熵的值就为0。

---

#### Generalization over Both Response Styles

接下来，作者使用 HH-RLHF 数据集的 chosen 部分进行训练，同时计算 rejected 部分的概率，实验结果如下图。

![d296ed923346ba3c741bac348b29ec84_MD5](https://picgo202.oss-cn-hangzhou.aliyuncs.com/d296ed923346ba3c741bac348b29ec84_MD5.png)

结果表明，单使用 chosen 部分进行训练，模型生成 rejected 数据的可能性与训练数据很接近。也就是说，同样的问题，模型的回复很有可能会是 rejected 数据。

原因也容易理解。无论是偏好对齐，还是逻辑间的微妙差别。前者出于某种偏好（流行的说法叫政治正确）上的区别，很有可能 rejected 回复就是模型该输出的事实本身。而后者的确是对模型的一种考验，但光凭 SFT 微调很难达到具备逻辑能力的目的。

（注：文中使用的 HH-RLHF 数据集，是 RLHF、 DPO 以及本篇论文提出的 ORPO 模型微调算法需要的数据集格式。该数据集格式包含一个问题，以及对应问题的 chosen 和 rejected 两个结果，用于模型训练。）

#### Penalizing Undesired Generations

为了解决损失函数不会学习 rejected 结果的问题，已经有论文做了类似的研究。比如，为了防止模型产生重复的 tokens，对已产生的 tokens 增加惩罚项，公式如下：
$$ (1-p_i^{(k)}) , k \in C_{recent}$$

### 4 Odds Ratio Preference Optimization

层层递进到这里，该进入论文的硬核部分了。

基于上述研究，论文提出了他们的偏好对齐算法：Odds Ratio Preference Optimization (ORPO)。

#### Odds Ratio

第一步定义 Odds Ratio 公式：
$$OR_θ(y_w, y_l)=\frac{odds_θP(y_w|x)}{oddsθP(y_l|x)}$$
其中：$odds_θ(y|x) = \frac{P_θ(y|x)}{1-P_θ(y|x)}$，$\log_2P_θ(y|x)=1/m\sum_{t=1}^m \log_2P_θ(y_t|x, y<t)$，$y_w$ -- chosen，$y_l$ -- rejected

$\log_2P_θ(y|x)$ 是输入为 x 的情况下，模型生成 y 回复的概率。

那么，$odds_θ(y|x)$ 就表示，生成 y 的概率是生成其他序列概率的倍数。 

最后，我们可以把 $OR_θ(y_w, y_l)$ 理解为模型生成 chosen 比生成 rejected 的可能性大多少。

#### Objective Function of ORPO

接下来定义 ORPO 损失函数：
$$L_{ORPO}=E_{(x,y_w,y_l)}[L_{SFT}+\lambda*L_{OR}]$$
其中，$L_{OR}=-\log_2σ (log \frac{odds_θP(y_w|x)}{oddsθP(y_l|x)} )$，$L_{SFT}$ -- 交叉熵损失

所以我们只需要关注 $L_{OR}$ 即可。最小化 ORPO 损失函数 $L_{ORPO}$, 就是在原有交叉熵损失下，最小化  $L_{OR}$ ，从而达到增加 rejected 的惩罚的目的。

#### Gradient of ORPO

进一步求解 ORPO 损失函数中 $L_{OR}$ 部分。求导过程如下：
$$\begin{align*} ∇θL_{OR} &= ∇θ \log_2σ \left( \frac{\log_2odds_θP(y_w|x)}{\log_2oddsθP(y_l|x)} \right) \\ &= σ'(\log_2g(x, y_l, y_w)) \cdot σ(\log_2g(x, y_l, y_w)) \cdot ∇θ \log_2g(x, y_l, y_w) \\ &= σ(-\log_2g(x, y_l, y_w)) \cdot ∇θ \log_2g(x, y_l, y_w) \\ &= \left( \frac{1 + odds_θP(y_w|x)}{odds_θP(y_l|x)} \right)^{-1} \cdot \left( ∇θ \log_2Pθ(y_w|x) \cdot \frac{1}{1 - P(y_w|x)} - ∇θ \log_2Pθ(y_l|x) \cdot \frac{1}{1 - P(y_l|x)} \right) \end{align*}$$

求导过程中需要注意：σ 是 sigmoid 函数，sigmoid 函数及导数的相关性质如下：
$S^{'}(x) = S(x)(1-S(x))$ 以及 $S(-x) = 1 - S(x)$

花了一点时间在 ChatGLM4 和 Kimi 的帮助下，才看明白求导过程。

从求导结果来看，前半部分中 rejected 概率（$odds_θ(y_l|x)$）越小，结果趋向于 0，模型参数就无需做太大更新，反之模型参数就需要大幅变化。这也反映了 ORPO 损失函数在 rejected 上的惩罚效果。

后续实验的设计、分析就不过多赘述。
### 后记

#### ChatGLM4 & Kimi

初读 ORPO 损失函数的求导过程时，抓耳挠腮，无从下手。 Google 也束手无策。

也闪过请 LLM 帮忙的念头，但信心不足，迟迟没有行动。最后抱着死马当作活马医的心态，试用了一把，一下就切中了要害。没有理解 σ 就是 sigmoid 函数的缘故。

试用了 ChatGLM4 和 Kimi，都给了正确的提示。 Prompt 设计如下：
> 1）上传此篇论文 PDF 文件；2）输入：“请详细解释 Appendix A 中 “A Derivation of ∇θLOR with Odds Ratio” 具体的推导过程”。

#### Llama3-Chinese-Chat
前几日，使用开源数据集，在 Llama3 基础上做了很简单的中文回答能力微调，效果很差。

同时也找了其他几个，同样一般。这些版本有个最大的问题，把 Llama3 原有的逻辑能力给微调消失了😅。

最终找到一个效果还不错的，推荐一下：[Llama3-8B-Chinese-Chat](https://huggingface.co/shenzhi-wang/Llama3-8B-Chinese-Chat)](https://huggingface.co/shenzhi-wang/Llama3-8B-Chinese-Chat)。

令人惊讶的是，前者（包括自己玩的）使用了 SFT + DPO 的微调方法，而 Llama3-8B-Chinese-Chat 采用了 ORPO 微调，因此就有了这篇解读文章。