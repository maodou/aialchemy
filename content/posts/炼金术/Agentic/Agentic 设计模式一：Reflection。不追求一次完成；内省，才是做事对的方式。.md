---
title: Agentic 设计模式一：Reflection。不追求一次完成；内省，才是做事对的方式。
summary: 
description: 
categories: Agentic
tags: 
externalUrl: 
date: 2024-07-10T11:43:40+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-08-22T22:44:48+08:00
share: true
---
[Agentic Design Patterns Part 2, Reflection](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-2-reflection/)，Andrew NG，deeplearning.ai

Agentic 是基于 LLM 的应用编程，区别于传统基于程序语言的编程方式。因此传统的设计模式也无法适用于 Agentic 应用编程。

在以 LLM 为 OS 之上开发应用，现今主流的设计模式是通过模拟人类处理具体任务的工作和组织形式来实现的。

Reflection 把它翻译为内省，是人类落地、完善一项任务的可行方案。小到一个方案的讨论（比如旅行安排），一次头脑风暴；大到一个互联网产品的开发，采用敏捷迭代的方式来精进或者即时转变产品；这些都是主流的任务落地方式。而背后的逻辑就是 reflection -- 内省，不断的优化迭代，而不是一次性设计执行。

文中以编程任务为例，阐释了此任务场景下的如何使用 reflection 设计模式，步骤如下：
1. 第一步不变，依旧需要清晰描述任务的提示词 -- “Please write code for {task}”。如敏捷产品开发，但敏捷不代表产品需求可以模棱两可和粗制滥造。
2. 接着，由另一个角色对现有的解决方式做出反馈。可以是同事 code review 的反馈，也可以是用户对产品的反馈，不同任务场景会有不同的角色参与。
3. 根据 2 中的反馈，不断完善任务结果。
4. 重复 2 、 3 步骤直到一个合适的出口。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240710155052.png)

Reflection 设计模式简单、直观，并且大部分应用已经这么在做了。模式介绍部分到此就差不多了，以下稍作展开。

LLM 使用人类的自然语言数据进行训练而来，处理任务的方式上已经和人类很相似了。它可能也会受意识形态（跟训练的数据有关）的影响，但是至少现在还没有 selfish 的成分，以及“屁股决定脑袋”的冲动。因此照理来说，光做事而言，LLM 比人类更具优势。

另一方面，也相互验证了 reflection 的必要性和有效性。这种模式可以进一步拓延到人类工作、生活的其他方面。不要期待一次性把事情做好，或者一夜暴富，相信一步一个脚印持续优化的复利效应。马斯克在星舰一声声的爆炸声中实践着这个原则。