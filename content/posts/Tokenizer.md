---
title: Tokenizer
summary: 
description: 
categories: 算法部分
tags: 
externalUrl: 
date: 2024-03-20T11:10:20+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-06-11T08:55:46+08:00
share: true
---
> 前言：此部分文章并非系统性的教学文章，网络上已经很多非常优秀的教学课程，顶尖且免费。比如，哔哩哔哩上李沐大神的《动手学深度学习v2》，Andrej Karpathy 在 YouTube 上教程，以及 Standford CS224N 课程。
> 这里主要记录作者的一些理解，有意思的知识点，或者豁然开朗的乐趣，希望你也能喜欢！
 
 > Tokenizer 没有很好的中文词汇来表达其含义，因此不做翻译，包括 token、tokenization。

如果你有代码层面使用开源的 LLM Pretrained 模型的经验，或者看过 HuggingFace 上各个模型使用的示例代码，那么应该不会对以下代码感到陌生。

起手第一行便是初始化 tokenizer，其后才是创建 model。

```
tokenizer = AutoTokenizer.from_pretrained("THUDM/chatglm3-6b", trust_remote_code=True)
model = AutoModel.from_pretrained("THUDM/chatglm3-6b", trust_remote_code=True).half().cuda()
```

看起来，tokenizer 和 model 之前相互独立。其实不然，tokenizer 和 model 之前是强绑定的，某个模型依赖一个特定的 tokenizer，而一个 tokenizer，比如 llama 系列的 tokenizer，往往可以支持同一个系列的衍生模型。

在系统层面，它们就完全属于两个不同的模块了。 Tokenizer 的历史要远早于基于 Transformer（2017 年）的大语言模型。Tokenizer 与所有机器学习算法类似，都需要训练。但与大语言模型的训练出来的结果不同，Tokenizer 与传统机器学习算法一样，对于相同的输入，输出是确定的；而大语言模型的惊人之处就在于，它更接近于人类，一个泛化的问题，不同的人，或者同个人在不同时间的回答会有区别。

接下来，我们会从1）什么是 Tokenizer？2）Tokenizer 相关算法。3）如何训练 Tokenizer？4）Tokenizer 的使用，这几方面来介绍 Tokenizer。

### 1. 什么是 Tokenizer？

狭义的大语言模型（除去图片、音、视频的多模态部分）是自然语言处理（NLP）的一种模型算法，如今也可以说是唯一事实模型标准。要做自然语言处理，无论是翻译，还是总结、补全、chat 都需要首先理解输入。

自然语言是人创造的，自然，而且也只能借鉴人是如何理解自然语言的。我们学习语言的时候，先学字（英文字母），再学词，最后成句变文章，当然无需纠结这个顺序是否严格。

字相对较少，特别是印欧语系，英文就 26 个字母（todo...）。而汉字字库将近十万，书面资料的 99% 的内容可被 3000 常用字覆盖。

词的数量就远不止字数了，收录在《牛津英语词典》中有 17 万余单词，派生词也有 9500，中文词汇更是达 40 多万，更不用提短语、句子了。

使用字还是词作为标准，各有优缺点。字、词少了，比较难学到“知识”。如果多了，模型参数膨胀，且对新词学习不利。

### 2. Tokenizer 算法

特别是英文，存在大量派生词，因此就有了 subword 的 tokenize 方式。

```
>>> tokenizer.tokenizer.sp_model.encode_as_pieces('Sentence Piece is an unsupervised text tokenizer and detokenizer mainly for Neural Network-based text generation systems where the vocabulary size is predetermined prior to the neural model training.')
# Chatglm3: AutoTokenizer.from_pretrained("THUDM/chatglm3-6b", trust_remote_code=True)
['▁Sent', 'ence', '▁P', 'iece', '▁is', '▁an', '▁uns', 'up', 'erv', 'ised', '▁text', '▁token', 'izer', '▁and', '▁det', 'oken', 'izer', '▁mainly', '▁for', '▁Ne', 'ural', '▁Network', '-', 'based', '▁text', '▁generation', '▁systems', '▁where', '▁the', '▁vocabulary', '▁size', '▁is', '▁pred', 'etermined', '▁prior', '▁to', '▁the', '▁neural', '▁model', '▁training', '.']
# Llama2: AutoTokenizer.from_pretrained("TheBloke/Llama-2-7b-Chat-GPTQ", use_fast=True)
['<s>', 'Sent', 'ence', 'Pie', 'ce', 'is', 'an', 'un', 'super', 'vised', 'text', 'token', 'izer', 'and', 'det', 'oken', 'izer', 'mainly', 'for', 'Ne', 'ural', 'Network', '-', 'based', 'text', 'generation', 'systems', 'where', 'the', 'voc', 'ab', 'ul', 'ary', 'size', 'is', 'pred', 'et', 'erm', 'ined', 'prior', 'to', 'the', 'neural', 'model', 'training', '.']
```

目前有三种主流的 Subword 算法，分别是 BPE 、Wordpiece 和 Unigram。

#### 2.1. BPE

Llama、GPT2 等模型使用 BPE 作为 Tokenizer 的算法。

以英文为例，绝大部分英文文字内容可用 ASCII 表中的 95 个可显字符表示。当然 BPE 算法处理的不仅仅是英文，处理的文本内容使用的是 Unicode-8 编码，后续会进一步展开。

1）BPE 算法首先会将所有英文字母、标点、空格等独立计算它们出现的频率。空格比较特殊，暂且不表。

2）使用贪心算法，每次取出现频率最高的相邻两个 subwords，组成新的 subword，同时更新涉及到的 3 个 subwords 的出现频率。

3）重复步骤二，直到满足自定义的 subword 个数为止。

此算法比较直观，使用了贪心算法，效率也很高。

```
def bpe_encode(text, vocab, vocab_scores):
    tokens = []

    # First encode every individual character in the input text
    for pos, char in enumerate(text):
        string = char
        id = str_lookup(string, vocab)
        if id == -1:
            print(f"not a good prompt at pos {pos}")
            sys.exit(1)
        tokens.append(id)

    # Merge the best consecutive pair each iteration, according to the scores in vocab_scores
    while True:
        best_score = -1e10
        best_id = -1
        best_idx = -1

        for i in range(len(tokens) - 1):
            # Check if we can merge the pair (tokens[i], tokens[i+1])
            # string = vocab[tokens[i]].rstrip(b'\x00') + vocab[tokens[i + 1]].rstrip(b'\x00')
            string = vocab[tokens[i]] + vocab[tokens[i + 1]]
            id = str_lookup(string, vocab)
            if id != -1 and vocab_scores[id] > best_score:
                # This merge pair exists in vocab! Record its score and position
                best_score = vocab_scores[id]
                best_id = id
                best_idx = i

        if best_idx == -1:
            break  # We couldn't find any more pairs to merge, so we're done

        # Merge the consecutive pair (best_idx, best_idx+1) into new token best_id
        tokens[best_idx] = best_id
        # Delete token at position best_idx+1, shift the entire sequence back 1
        tokens = tokens[0:best_idx + 1] + tokens[best_idx + 2:]

    return tokens
```

#### 2.2. Wordpiece

BERT 模型使用了 Wordpiece 算法。它与 BPE 算法非常类似，都基于合并（merge）规则的算法，区别于 Unigram 算法。

Wordpiece 算法的逻辑基本与 BPE 算法类似，使用最大似然估计（互信息）算法替换了贪心算法。

以下对最大似然估计算法做个简要介绍。

对于文本训练集 **S = (t1, t2, ... , tn)** 由 n 个 subword 组成，对于英文而言初始化的 subword 就是 ASCII 表中的 95 个可显字符。假设各个 subword 是独立存在的，那么训练集 S 的似然值等价于所有 subword 的概率乘积：

![](https://cdn.nlark.com/yuque/__latex/35b951dadb13873a50212f46a2920adf.svg)