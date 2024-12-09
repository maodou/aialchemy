---
title: LoRAX -- 以一当百
summary: 
description: 
categories: 致敬MP
tags: 
externalUrl: 
date: 2024-05-05T21:30:01+08:00
draft: false
showauthor: true
authors:
  - 蚂蚁无双
featureimage: 
lastmod: 2024-05-11T02:24:05+08:00
share: true
---
[# LoRA Exchange (LoRAX): Serve 100s of Fine-Tuned LLMs for the Cost of 1](https://predibase.com/blog/lora-exchange-lorax-serve-100s-of-fine-tuned-llms-for-the-cost-of-one)

LoRAX 是一个用于降低基于同一基础模型（base model) 的多个 LoRA 微调模型的推理成本，并同时加速推理的 LLM 引擎。

这里的关键是，使用 LLM 的业务场景或者产品，需要用到**多个 LoRA 微调模型**。

![](https://picgo202.oss-cn-hangzhou.aliyuncs.com/399fef6241fe0d2f2d0a9f18fdd1fbbc_MD5.png)

如图所示，这些 LoRA 微调模型由同一个基础模型微调而来，不然不可能共用基础模型的 weights。

### Fine-Tuning and Serving LLMs with LoRA
> To make fine-tuning less resource-hungry, _parameter-efficient fine-tuning_ techniques like Low Rank Adaptation (LoRA) introduce _adapters_ consisting of a small number of new parameters that are trained, while the original model parameters remain frozen.

首先简要介绍了 LoRA 微调方法，以及它的好处：占用更少的资源（计算、VRAM、磁盘等资源），获取同等的性能。

LoRA 微调后，模型 weights 可以分为，base model weights (图中黄色柱状部分) 和 adapter weights (图中蓝、绿、红柱状部分)，而 adapter weights 往往只占 10%。

> The downside of this is that if multiple models are fine-tuned with LoRA, each needs to be deployed together with the original LLM on a dedicated set of resources, which can quickly add up.

如果有这么一个应用场景，当你发现一个比较好的开源基础模型，比如 Llama3。你想基于这个模型，结合不同的数据集，面向不同的客户群体，提供差异化的微调模型。一种传统的部署方式，如上图左边部分所示。

这种部署方式的缺点可想而知，随着微调模型的增加，成本会线性上升。特别是在模型较小，用不完一个部署单元的 VRAM，且不同微调模型的访问差异较大，总体访问量又不大的场景，LoRAX 给了一个省钱的方案。

### Introducing LoRA Exchange (LoRAX)
> 1. **Dynamic Adapter Loading**, allowing each set of fine-tuned LoRA weights to be loaded from storage just-in-time as requests come in at runtime, without blocking concurrent requests.
> 2. **Tiered Weight Caching**, to support fast exchanging of LoRA adapters between requests, and offloading of adapter weights to CPU and disk to avoid out-of-memory errors.
> 3. **Continuous Multi-Adapter Batching**, a fair scheduling policy for optimizing aggregate throughput of the system that extends the popular continuous batching strategy to work across multiple sets of LoRA adapters in parallel.

方案其实也挺简单明了的。如果你对操作系统提供的动态库有所了解，LoRAX 方案就非常容易理解了。不同微调模型就像不同的应用，共用同一个 base model weights 库。

#### Dynamic Adapter Loading
> Unlike conventional serving infrastructure that preloads all model weights during initialization, LoRAX only loads the pretrained base LLM weights during initialization, and dynamically loads each set of fine-tuned LoRA adapters just-in-time at runtime.

LoRAX 的部署如上图右边部分。不同微调模型的 adapter weights 是在运行时加载的。

> To avoid blocking ongoing requests from other users, the LoRAX system maintains an individual _request queue_ per fine-tuned adapter.

为了不阻塞来自不同用户对不同微调模型的请求，LoRAX 为每个微调模型维护了一个独立的 request queue。这也是我们在做系统研发时，通过 queue 模式，异步承接大量用户请求，提高吞吐量的惯用手段。
>
>In practice, we’ve observed the overhead of dynamically loading in a new adapter to be on the order of 200ms

不同 adapter 上下文切换大概会有 200ms 的额外开销，对于 LLM 来说可能是在可接受范围之内。

#### Tiered Weight Caching
> As more fine-tuned models are loaded into a single LLM deployment, the memory overhead increases. To avoid encountering an _out of memory error (OOM)_, the LoRAX system implements a _tiered weight caching_ strategy that offloads adapter weights from GPU → CPU → disk to trade off between adapter exchange latency and memory overhead.

Adapter weights 多层缓存，GPU → CPU → disk。

> When an adapter does need to be evicted from the GPU, we transition it to CPU (host memory) using a _least-recently used (LRU)_ policy.

多层缓存，缓存 LRU 算法，对于大部分开发者来说是家常便饭，不多赘述。

> Putting all of this together allows you to pack upwards of 100 models into a single deployment without the need to scale up to additional replicas barring high request volume.

无疑，adapters 需要占用的内存，多到 CPU 的内存都放不下，要放到磁盘时，这个性能级别就无法接受了。

#### Continuous Multi-Adapter Batching
> One of the most important techniques for enabling high throughput text generation has been [continuous batching](https://www.usenix.org/conference/osdi22/presentation/yu), whereby multiple requests can be dynamically batched together between each token generation _step_ as new requests come in and old requests complete.

LoRAX 使用了一种增加吞吐量的 continuous batching 技术 -- [Orca](https://www.usenix.org/conference/osdi22/presentation/yu)。

Orca 大致思想如下：基于 Transformer 架构的 LLM 模型本身就能够做 batch 处理，但是 query input 维度必须相同，且只有同个 batch 的请求都处理完才会返回。Orca 采用一种新的  iteration-level 调度，解决上述两个问题。

LoRAX 在 Orca 基础上还需要解决 multi-adapter 问题。

> 1. At any given time, some number of adapters N (limited by GPU memory) will be marked as “active”, with their weights loaded onto the GPU and available for use during decoding.
> 2. Requests from activate adapters will be drained from their respective queues and batched together continuously. A simple mask ensures that the correct adapter is applied to each request in the batch when computing the activations for each layer (see figure below).
> 3. After a configurable amount of time has elapsed, the scheduling system will move to the next set of adapters in a round robin fashion. In practice, this means the adapter that has been in the active set the longest will be evicted, and the adapter with non-empty request queue that has been waiting the longest will become active. The amount of time to wait before exchanging active adapters can be increased to prioritize throughput, or decreased to prioritize latency.

![Decoding using multiple adapters within a single batch. Masks ensure that only the right adapter is used for processing each element of the batch.](https://picgo202.oss-cn-hangzhou.aliyuncs.com/7b2a34b1d4b63e3849e58bf42c903bce_MD5.png)

其实，如果了解操作系统的线程调度机制，或者像 Go、 Java 语言等提供的协程调度机制，那么这里的 multi-adapter 的调度就小儿科多了。

其他不多做介绍，看原文也很容易理解。此处用到 mask 来控制不同 adapter weights 进行计算操作，倒是可以借鉴一下。

略去 LoRAX 的使用部分。

### Closing Thoughts

我们直接来看结束语。

> we believe that the future is fine-tuned, specialized LLMs for your tasks.

LoRAX 有价值的重中之重，还是要基于这么一个假设：垂直的微调模型是 LLM 的未来。

> - Speeding up inference with smarter, task-specific decoding.
> - Extending the context length of models to handle very long input sequences.
> - Fine-tuning with as few as 10 to 100 examples.

列举了 LoRAX 后续的三个方向，前两个会逐步改善，业界已经有了不少的方法。至于第三个嘛？拭目以待！