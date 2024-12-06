

### 模型选择

#### 通用 LLM
原则：
- 与电子类产品相同，或者更加激进 -- 同一系列模型，用新不用旧
- 绝对不能光看 benchmark！
- 非商用模型，language-native 还是比较重要的。
- 不要对 finetune 抱太多希望，前提是需要相对完整的模型开发能力。
- 一般来说，参数与能力成正比
- License 

候选模型：
- Llama3.2-3B / Llama3.1-8B 
	非国产，对中文支持一般，特别是在回复上，会不自然使用英文。支持 128k 上下文。
	LLAMA 3.1 COMMUNITY LICENSE AGREEMENT，Your use of the Llama Materials must comply with applicable laws and regulations
- Ministral-8B-Instruct-2410
	非国产，有 License 风险，支持 128k 上下文。
	Mistral AI Research License， You shall only use the Mistral Models, Derivatives (whether or not created by Mistral AI) and Outputs for Research Purposes.
- Qwen2.5-7B-Instruct
	国产，阿里巴巴，支持 128k 上下文。Apache license 2.0
- GLM-4-9B-chat
	国产，清华 智普，支持 128k 上下文。自己的 license，商用需要在 https://open.bigmodel.cn/mla/form 上做登记。

![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20241118094051.png)


![image.png](https://picgo202.oss-cn-hangzhou.aliyuncs.com/20241118102117.png)

#### 场景
报表解读
运营问题检索