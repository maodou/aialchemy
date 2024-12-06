[Agentic Design Patterns Part 4, Planning](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-4-planning/?ref=dl-staging-website.ghost.io)，Andrew NG，deeplearning.ai

就在去年（好像过去了很久），很多人都已经感受过“ChatGPT moment”。

“ChatGPT moment”形容的非常贴切，初次见面不难不让人惊艳。但对于大部分旁观者而言，moment 终究难以持续。随着对各种实际任务的尝试，怯魅的过程也很迅速，看起来 LLM 也没有那么灵光了。

陷入如此情景实属正常，毕竟 LLM 也还在萌芽期，为大众所认知也不过 1 、 2 年，还有很大的发展潜力。另一个不可忽视的原因，很多“用户”都不够资深，更重要的是很多“产品”（Agent）还不够傻瓜，依旧停留在简单的对话框阶段。因此，Agentic 设计模式的重要性就更加不言而喻了。到这里，不得不再次推荐百度[橙篇](https://cp.baidu.com/)。

文中给出了另外一种 moment，“AI Agentic moment”。Andrew 的 AI Agentic moment 如下文所述：

> 我曾多次私下测试这个代理，在测试过程中，它会始终使用网络搜索工具收集信息并撰写摘要。然而，在实况演示中，网络搜索API意外地返回了速率限制错误。当我以为我的演示即将在公众面前失败，并为接下来将会发生的事感到恐惧时，令我惊讶的事发生了：代理巧妙地转向了一个维基百科搜索工具——我忘记了我曾经给过它——并使用维基百科而不是网络搜索完成了任务。
> （翻译自 [Kimi](https://kimi.moonshot.cn/)，做了部分的调整）

很多实际工作很难通过一个简单的提示词描述清楚，往往需要 Divide and Conquer 方式才能更好的解决。 Divide and Conquer 是一种计算机编程思想，对应的人类的处理方式就是 planning（规划）。

将一个现实任务规划成、分解为一个个能够独立执行的子任务，并以适当的形式串联起来得到最终想要的结果，这种处理方式同样适用于 LLM。

规划的概念易于理解，不多过举例赘述。但需要注意的是，Planning 设计模式与 [Reflection](https://mp.weixin.qq.com/s/wykc0X22Rm9oJGBfOxNjWg) 和 [Tool Use]() 在稳定性上还有一定差距。按照 Andrew 的使用经验，后两者现在能做到相对稳定的输出结果；而 Planning 的结果偏差相对会大许多。

这个结果也不怎么让人惊讶。首先，Planning 让执行流程变长，任务需要先生成子任务，子任务各自完成后还需进一步整合。其次，生成的子任务就有不确定性，这大大增加了结果的不确定性。（虽然 reflection 流程也比较长，但没有子任务的烦恼）。最后，Planning 还在探索期，需要一个成熟的过程。

Keep learning!




