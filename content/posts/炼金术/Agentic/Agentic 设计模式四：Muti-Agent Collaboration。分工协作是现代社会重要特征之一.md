---
title: Agentic 设计模式四：Muti-Agent Collaboration。分工协作是现代社会重要特征之一
summary: 
description: 
categories: Agentic
tags: 
externalUrl: 
date: 2024-07-12T13:40:03+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-08-22T22:44:48+08:00
share: true
---
[Agentic Design Patterns Part 5, Multi-Agent Collaboration](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-5-multi-agent-collaboration/?ref=dl-staging-website.ghost.io)，Andrew NG，deeplearning.ai

Mult-Agent Collaboration（多角色协作）是 Agentic 设计模式系列的最后一个设计模式。但可以肯定的是，随着 LLM Agentic 应用的大规模研发和普及，会有大量其他 Agentic 设计模式涌现出来。

Muti-Agent Collaboration 与 [Planning](https://mp.weixin.qq.com/s/Bg1gN-_666olew57qsBo0g) 设计模式非常类似，都借用了 Divide and Conquer 的理念。但它们之间的区别可以通过面向过程编程与面向对象编程的差别来理解。

Planning 设计模式就像用编程语言实现一个算法，我们会将算法逻辑分解成各个子过程，每个过程都通过不同的函数实现，并在一个主入口函数中集成。而 Muti-Agent Collaboration 设计模式更像处理业务逻辑，需要抽象成不同的对象，各自完成特定的子任务，最终协同完成整个目标任务。

简而言之，Muti-Agent Collaboration 设计模式，通过模拟人类在处理业务任务时所采用的不同角色和组织形态来实现任务目标。随着技术的发展，这种设计模式的变化和创新将是持续且必然的。

多角色协作模式的基本流程如下：
1. 确定基本的业务流程和相应的角色（Agent）。
2. 为每个角色设计合理的提示词。
3. “用户”任务请求作为流程的入口，驱动业务流转。
4. 按照流程设计，不同角色使用不同的提示词，并在角色间流转。
5. 在流程中会设计一定的出口条件，满足对应条件或者流程流转完毕，流程结束。
6. 具体任务成果会在流转中生成。

下图以软件工程为例，演示了 Muti-Agent Collaboration 设计模式是如何组织不同角色进行软件开发的：
![多角色协作设计模式，软件工程示例](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240712141928.png)

对于一个程序员而言，应该马上能嗅到其中能够被框架化的可能。不出意外，已经有人在做相关的工作了。这里我们来简单介绍一个叫虚拟软件公司的开源项目 **[ChatDev](https://github.com/OpenBMB/ChatDev)**。 

该项目提供了以下思路值得借鉴：
1. 设计了一套类似于工作流引擎的流程链 ChatChain，来模拟软件开发流程。
2. 并且可以通过配置的方式进行流程、阶段（交互形式等）、角色（提示词等）的定义。对应下图中的三个 json 文件。
![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240712143653.png)

该项目专门针对软件工作领域的配置设计，并没有实现跨领域的兼容性。目前，项目功能存在一些限制，比如只支持ChatGPT不同版本的API调用。尽管如此，我们只要对源代码通过简单的修改，该项目可以适配其他供应商的API，包括本地模型。

需要用到多角色协作设计模式的任务，复杂度相对较高。这不仅对 LLM 自身的性能有极高的要求外，也对一些技术性指标有严格的标准，例如LLM支持的最大输入和输出 token 数。

以软件开发为例，任务提示词往往需要包含大量的上下文信息，且所需生成的代码量通常会超出现有 LLM 支持的最大 token 数（4k？）。因此，这需要进行大量的工程优化和适配工作。随着 LLM 模型能力的提升，一些现有问题可能会得到解决。然而，工程技术的运用是不可避免的，除非将来 AI 技术足够强大，不再需要额外的辅助工具。

Keep learning!