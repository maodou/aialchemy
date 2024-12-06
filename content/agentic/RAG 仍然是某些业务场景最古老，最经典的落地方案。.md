本文主要参考了论文《Searching for Best Practices in Retrieval-Augmented Generation》，以及其他几篇论文和网络上相关资料。以下以此论文为主线介绍 RAG 最佳实践。

先将结论放在最前面。

## 总结

1. Query Classification，使用产品交互层面解决，去掉此步，减少出错率和延迟。
2. Retrieval，Chunking 采用：句子级别 Chunking，chunk 大小 175 ~ 512 tokens，overlap 大小为 20 的滑动窗口技巧；向量数据库有条件使用 Milvus，快速应对 Chroma 也未尝不可；Retrieval 方法建议采用预生成文档+混合搜索（Hybrid Search with HyDE）的模式，其中预生成文档只要一份即可，α 取值 0.3；Embedding 模型建议 bge-m3。
3. Reranking，模型建议配合 Embedding 模型采用  bge-reranker-v2-m3。
4. Repacking，同一文档的 chunks 需要按先后顺序排，其他到底采用 forward、reverse 还是 sides 可以做不同尝试。
5. Summarization，使用限制条件下性能最好的 LLM。
## Searching for Best Practices in Retrieval-Augmented Generation, Xiaohua Wang ..., Fudan University

文如其名，这是一篇介绍 RAG 最佳实践的文章，并以下图总览展开。文章剩余部分基本上就是对 RAG 的整体流程以及各个部件的阐释。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921171715.png)

论文对 RAG 描述可谓面面俱到，给出了各个部件在当时（或者作者实验而来）的最佳实践，并贴心的通过“蓝色”字体表明。但需要各位观众或者接下来要采用 RAG 的实践者注意的是：AI 领域日新月异，各部件的最佳实践会随时更新，而局部的优化可能会进一步影响到整体流程。 

### 总体流程

1. Query Classification，首先判断此次请求是否有必要进一步获取相关信息，还是通过 LLM 直接回复即可。
2. Retrieval，向量化 query 后，从向量数据库通过相似度算法，获取相关的文档 chunks。
3. Reranking，使用 LLM（小）对文档 chunks 打分。
4. Repacking，依据 reranking 后的排名（得分），对 chunks 进行筛选，并作为组织 prompt 的依据。
5. Summarization，prompt 输入到通用 LLM（大）给出总结。

其中 Query classification 没有过多关注。Retrieval 部分，有大量细节可以借鉴。 Reranking 不在经典 RAG 流程中，它增加了一定的响应时间消耗，但普遍认为效果不错。Repacking，提示词工程师的工作，以后我们都是提示词工程师！Summarization，做好你能做的，剩下的都交给 LLM。

### Query Classification

先对请求进行分类，那么它是针对通用 Agent 的。也就是说，基于 LLM 开发的应用，其功能不止应用于需要获取内部资料的场景。这就会导致两个问题：

1. Query classification 导致的分类错误的问题，虽然论文中号称他们训练的模型有 90%+ 的准确率。首先在真实业务场景中，对 90%+ 的结论是存在疑问的。另一方面，即使达到如此的准确率，因为这只是 RAG 中的其中一步，而且对于 RAG 方案而言是一个串联概率，相乘后对总体性能影响是不容忽视的。
2. 通用 Agent 的实现对 prompt 的组织的也是一种挑战。

幸好如今市面很多大模型应用已经给出了一个不算完美的解决方案，通过在产品交互层面改进 -- 不同场景对应不同 Agent（机器人）的形式，来解决这部分的问题。显然这种方式也有其弊端，那么是不是以后我们在这之上还要有个 Agent 搜索引擎？这就留待后续解决吧。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921224719.png)

### Retrieval

涉及文档 retrieval 相关技术需要关注的细节就更多了。

#### Chunking
内部资料文档内容一般较长，而向量化能够支持的 token 长度在 4096，embedding 后大小在 1024 左右。因此需要对长文档做 chunking 操作。Chunking 操作又可以分为以下几类：

1. Token 级别的 Chunking，这个也是做简单的，就是固定长度。这就可能会截断文档中完整的句子和段落。
2. 语义级别的 Chunking，使用 LLM 来截取不同的 chunks。显然这个成本相当高。
3. 句子级别的 Chunking，保持一定的语义连贯性，成本也相对较低。

**论文采用了句子级别的 Chunking。**

其次，每个 chunks 的大小也很重要，论文对不同 chunk 大小做了比较如下图。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921232935.png)

最后，还有一些 chunk 技巧可以使用，论文中也做了相关对比。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921233259.png)

**最终，论文采用：句子级别 Chunking，chunk 大小 175 ~ 512 tokens，overlap 大小为 20 的滑动窗口技巧。** 这部分我们完全能够直接借鉴使用。

#### Embedding 模型

论文比较了当时几个主流的 Embedding 模型，最终使用了 LLM-Embedder。最近一年 BAAI 又更新了不少版本，对多语言（特别是中英文的支持）而言可以尝试一下它新出的 bge-m3 模型。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921233753.png)

#### 向量数据库

论文采用 Milvus。如果数据量不大，需要快速上手，使用 Chroma 也不是不能选。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240921234221.png)

#### Retrieval 方法

使用者往往没有经过 prompt 培训，他们给出的 query 一般会比较简陋。那么如何通过简陋的 query 输入获得合适的 chunks，也存在不同的 Retrieval 方法：

1. Query 重写，使用 LLM 来更好的组织 query。
2. Query 分解，将原生 query 分解成各个子问题。
3. 预生成文档，根据用户 query，给出对应的回复，也就是这里的预生成文档。

上述三种方法增强过的 query，最后都要通过向量数据库去获取相关的 chunks。同时向量数据库也提供了不同的相似度算法，论文使用了 BM25 -- sparse retrieval 算法，和 Contriever -- dense retrieval 算法。也可将两者同时使用，并通过 α 超参数来调节两者的权重，这就是混合搜索（Hybrid Search）。
公式：Sh = α · Ss + Sd

综上，论文建议采用预生成文档+混合搜索（Hybrid Search with HyDE）的模式，其中预生成文档只要一份即可，α 取值 0.3 效果最好。

### Reranking

无论使用哪个相似度算法，或者混合搜索的模式，获取得到的 chunks，依旧会有数量可观的一部分文档不符合要求。因此需要 Reranking 来对获取到的 chunks 进行进一步的打分删选和排序。

这里就不采纳论文中的结论，使用较新的数据。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240918112519.png)

鉴于性能、延迟等因素的综合考量，推荐使用 bge-m3 和 bge-reranker-v2-m3 的组合。

### Repacking

Repacking，其实就是根据 Reranking 后删选出的 chunks 各自得分，顺排（forward）、逆排（reverse）和两端排（sides）。论文采用了 sides 两端排，依据是有篇论文中得出在相关文档头和尾部分的内容对 LLM 的影响最大。

另一篇论文《In Defense of RAG in the Era of Long-Context Language Models》提供了一个新的角度，它增加另一个更重要的纬度，同一文档需要按先后顺序排，最后的效果会更好。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20240922005402.png)


### Summarization

建议使用市面最好的模型，ChatGPT4 和 Claude3.5。由于各种原因无法使用的，国内 kimi 、 glm4 对于 RAG 场景的应用也都好不错。最后需要内网部署，GPU 有限的，GLM4-9b 会是一个不错的选择。
