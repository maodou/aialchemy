---
title: Agentic 设计模式二：Tool Use。制造和使用工具，是区分人与动物的重要标志。
summary: 
description: 
categories: Agentic
tags: 
externalUrl: 
date: 2024-07-10T22:47:25+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-12-09T15:29:30+08:00
share: true
---

[Agentic Design Patterns Part 3, Tool Use](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-3-tool-use/?ref=dl-staging-website.ghost.io), Andrew NG, deeplearning.ai

LLM 脱胎于人类自然语言，却天然受困于虚拟的二进制环境。想要充分发挥它的能力，必须得让他使用工具。但其中蕴含的潜在危险性，不在本文的讨论范围之内。

市面上很多主流商用 LLM 都或多或少的提供了工具使用的能力，包括文件解析，网页搜索等。（注：我们这里把图片识别视为模型本身的多模态能力，不是工具。）

国产大模型也不落后，特别是 [Kimi](https://kimi.moonshot.cn/) 在网页搜索上的性能体验，不落下风。除了 Kimi 外，[Chatglm](https://chatglm.cn/main/alltoolsdetail)处理逻辑类任务（比如代码相关），[秘塔 AI 搜索](https://metaso.cn/)基于搜索的应用，对标国外的 Perplexity，都有不错的使用体验。百度[橙篇](https://cp.baidu.com/chat?type=compose)的长文写作，个人认为是这个没落的技术大厂，不多的能拿得出手的 LLM 应用。

理论上，人类迄今能使用的二进制工具，LLM 都能够使用。即便是物理世界的工具，也可以通过二进制手段间接使用。这也是现在 LLM 在 ChatGPT 发布函数调用功能以来，工具使用方面的发展方向。

内容搜索工具的使用，大家在 Kimi 直接体验就行，很直观。

另外一部分看起来对现有计算机不足一提的能力，比如稍复杂四则运算，LLM 却很难给出正确答案。其实人不借助计算器或者 Python 等编程语言，也很难做到准确计算。

文中举了两个例子如下图所示，分别展示了 Bing CoPilot 的内容搜索，以及 ChatGPT 的代码执行器的工具使用功能。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240711221523.png)

目前为止 LLM 使用工具的流程实现，大致如下图所示：

![Tool Use 流程图](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240711233632.png)

从上图可知，LLM tool use 模式需要一定的 Agent 应用工程能力，以及精巧的提示词设计。更重要的是 LLM 本身需要具备 tool use 的能力。

如果想尝试一下 tool use 模式开发，推荐国产 ChatGLM。

Keep learning!
