- [A High-level Overview of Large Language Models](https://www.borealisai.com/research-blogs/a-high-level-overview-of-large-language-models/)
- [AI 研发提效的正确姿势：开源 LLM + LoRA](https://zhuanlan.zhihu.com/p/620236884)

# 预训练 Pre-training
- [Improving Language Understanding by Generative Pre-Training](https://s3-us-west-2.amazonaws.com/openai-assets/research-covers/language-unsupervised/language_understanding_paper.pdf)

# 指令精调 Instruction fine-tuning
- [Finetuned Language Models Are Zero-Shot Learners](https://arxiv.org/abs/2109.01652)

## Self Instruct
- [SELF-INSTRUCT: Aligning Language Models with Self-Generated Instructions](https://arxiv.org/pdf/2212.10560.pdf)

## CoT
- [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/pdf/2201.11903.pdf)
   - 单一LLM + 针对性的提示词 ==> 可以解决所有语言推理任务

## ToT
- https://arxiv.org/pdf/2305.10601.pdf
- [思维树/TOT（Tree of Thoughts），让大型语言模型深思熟虑的解决问题](https://zhuanlan.zhihu.com/p/634180290) | [Tree of Thoughts:利用大语言模型解决复杂的推理问题](https://zhuanlan.zhihu.com/p/631570761)
- [Official Implementation of "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"](https://github.com/princeton-nlp/tree-of-thought-llm)
- https://github.com/kyegomez/tree-of-thoughts
   - Plug in and Play Implementation of Tree of Thoughts: Deliberate Problem Solving with Large Language Models that Elevates Model Reasoning by atleast 70%

## 参考
- [**提示工程指南**](https://www.promptingguide.ai/zh)
- [了解如何准备用于微调的数据集](https://learn.microsoft.com/zh-cn/azure/ai-services/openai/how-to/prepare-dataset)
- [中文医学大模型“本草”（原名华驼）：医学知识增强在中文大型语言模型指令微调上的初步探索](https://www.kuxai.com/article/1113)
- [指令精调llama-7b](https://zhuanlan.zhihu.com/p/616504594)
- [Tree of Thoughts (ToT)](https://www.promptingguide.ai/techniques/tot) | [Chain-of-Thought Prompting](https://www.promptingguide.ai/techniques/cot)
- [扩展指令微调语言模型](https://www.promptingguide.ai/zh/models/flan)

# 基于人工反馈的强化学习 Reinforcement learning from human feedback (RLHF)
- [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)

# 常用微调方法
## Adapter Tuning
2019年谷歌的研究人员首次在论文《Parameter-Efficient Transfer Learning for NLP》提出针对 BERT 的 PEFT微调方式，拉开了 PEFT 研究的序幕。他们指出，在面对特定的下游任务时，如果进行 Full-Fintuning（即预训练模型中的所有参数都进行微调），太过低效；而如果采用固定预训练模型的某些层，只微调接近下游任务的那几层参数，又难以达到较好的效果。于是他们设计了 Adapter 结构，将其嵌入 Transformer 的结构里面，在训练时，固定住原来预训练模型的参数不变，只对新增的 Adapter 结构进行微调，保证训练的高效性（也就是尽可能少的引入更多参数）。从实验结果来看，该方法能够在只额外对增加的 3.6% 参数规模（相比原来预训练模型的参数量）的情况下取得和Full-Finetuning 接近的效果（GLUE指标在0.4%以内）。

## Prefix Tuning
2021年斯坦福的研究人员在论文《Prefix-Tuning: Optimizing Continuous Prompts for Generation》中提出了 Prefix Tuning 方法。与Full-finetuning 更新所有参数的方式不同，该方法是在输入 token 之前构造一段任务相关的 virtual tokens 作为 Prefix，然后训练的时候只更新 Prefix 部分的参数，而 Transformer 中的其他部分参数固定。该方法其实和构造 Prompt 类似，只是 Prompt 是人为构造的“显式”的提示，并且无法更新参数，而Prefix 则是可以学习的“隐式”的提示。

## Prompt Tuning
Prompt Tuning 是2021年谷歌在论文《The Power of Scale for Parameter-Efficient Prompt Tuning》中提出的微调方法。该方法可以看作是 Prefix Tuning 的简化版本，只在输入层加入 prompt tokens，并不需要加入 MLP 进行调整来解决难训练的问题，主要在 T5 预训练模型上做实验。似乎只要预训练模型足够强大，其他的一切都不是问题。作者也做实验说明随着预训练模型参数量的增加，Prompt Tuning的方法会逼近 Fine-tune 的结果。
- Prompt 长度影响：模型参数达到一定量级时，Prompt 长度为1也能达到不错的效果，Prompt 长度为20就能达到极好效果。
- Prompt初始化方式影响：Random Uniform 方式明显弱于其他两种，但是当模型参数达到一定量级，这种差异也不复存在。
- 预训练的方式：LM Adaptation 的方式效果好，但是当模型达到一定规模，差异又几乎没有了。
- 微调步数影响：模型参数较小时，步数越多，效果越好。同样随着模型参数达到一定规模，zero shot 也能取得不错效果。
- 当参数达到100亿规模与全参数微调方式效果无异。

## P-Tuning v1
P-Tuning 方法的提出主要是为了解决这样一个问题：大模型的 Prompt 构造方式严重影响下游任务的效果。P-Tuning 提出将 Prompt 转换为可以学习的 Embedding 层，提出用 MLP + LSTM 的方式来对 prompt embedding 进行一层处理。P-Tuning 和 Prefix-Tuning 差不多同时提出，做法其实也有一些相似之处，主要区别在：
- Prefix Tuning 是将额外的 embedding 加在开头，看起来更像是模仿 Instruction 指令；而 P-Tuning 的位置则不固定。
- Prefix Tuning 通过在每个 Attention 层都加入 Prefix Embedding 来增加额外的参数，通过 MLP 来初始化；而 P-Tuning 只是在输入的时候加入 Embedding，并通过 LSTM + MLP 来初始化。

## P-Tuning v2
P-Tuning 的问题是在小参数量模型上表现差，《P-Tuning v2: Prompt Tuning Can Be Comparable to Fine-tuning Universally Across Scales and Tasks》提出的 P-Tuning v2 的目标就是要让 Prompt Tuning 能够在不同参数规模的预训练模型、针对不同下游任务的结果上都达到匹敌 Fine-tuning 的结果：
- 不同预训练模型大小下的表现，在小模型下取得与 Full-finetuning 相近的结果，并远远优于 P-Tuning。
- 不同任务下的 P-Tuning v2 效果都很好，而 P-Tuning 和 Prompt Learning 效果不好；同时，采用多任务学习的方式能在多数任务上取得最好的结果。

## AdaLoRA
预训练语言模型中的不同权重参数对下游任务的贡献是不同的。因此需要更加智能地分配参数预算，以便在微调过程中更加高效地更新那些对模型性能贡献较大的参数。通过奇异值分解将权重矩阵分解为增量矩阵，并根据新的重要性度量动态地调整每个增量矩阵中奇异值的大小。这样可以使得在微调过程中只更新那些对模型性能贡献较大或必要的参数，从而提高了模型性能和参数效率。

## PETL的通用形式
通过对 Prefix Tuning 的推导，得出了和 Adapter Tuning 以及 LoRA 形式一致的形式。

# LoRA
基于大模型的内在低秩特性，增加旁路矩阵来模拟全参数微调，LoRA 通过简单有效的方案来达成轻量微调的目的。

参考：
- [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685)
- [一文读懂：LoRA实现大模型LLM微调](https://blog.51cto.com/u_15668366/6504793)
- [能像乐高一样组合，LoraHub挖掘LoRA 模块化特性](https://www.kuxai.com/article/1302)
- [**使用PaddleNLP训练Lora教ChatGLM-6B作数学题，具体步骤及效果测试，A100是个好东西**](https://zhuanlan.zhihu.com/p/632001583)
- [利用LoRA对LLM进行参数高效的微调](https://zhuanlan.zhihu.com/p/632159261)
- [baichuan-7B 模型使用/训练/Lora/测评](https://zhuanlan.zhihu.com/p/637343740)
- [alpaca-lora](https://github.com/tloen/alpaca-lora) Instruct-tune LLaMA on consumer hardware
- [【开源GPT】骆驼语言团队进一步开源“驼铃”，单张显卡1小时训练属于你自己的中文语言模型](https://zhuanlan.zhihu.com/p/616784584)
   - https://github.com/LC1332/Luotuo-Chinese-LLM
- [LORA：大模型轻量级微调](https://zhuanlan.zhihu.com/p/623543497)
- [大模型微调总结](https://zhuanlan.zhihu.com/p/627642632)

## QLoRA
- [QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314)
- [微调百川Baichuan-13B保姆式教程，手把手教你训练百亿大模型](https://zhuanlan.zhihu.com/p/643950663)

## GLoRA
- [One-for-All: Generalized LoRA for Parameter-Efficient Fine-tuning](https://arxiv.org/pdf/2306.07967.pdf)广义的参数高效微调方法


# In-context Learning（ICL）
- [In-context Learning - A New Paradigm in NLP?](https://nlplab.tech/blog/in-context-learning-a-new-paradigm-in-nlp/)

## 上下文指令学习（ICIL）
- [ ] [In-Context Instruction Learning](https://arxiv.org/pdf/2302.14691.pdf)

## 多任务提示微调（MT）
- [ ] [Exploring the Benefits of Training Expert Language Models over Instruction Tuning](https://arxiv.org/pdf/2302.03202.pdf)

# 实例
## 问答微调
[Fine-Tune Transformer Models For Question Answering On Custom Data](https://www.youtube.com/watch?v=KO44Tq8DmTQ)
1. 自有数据集，对LLM进行微调
2. deepset/reberta-base-squard2
3. 数据集SubjQA 10k问答数据
4. 数据集预处理 training / validation / compute_metrics
5. AutoModelForQuestionAnswering / TrainingArguements / Trainer.train()
6. Inference / pipeline("question-answering", model=...)
7. 将生成的模型上传到HuggingFaceHub

## 生成Midjourney提示词
- ["okay, but I want GPT to perform 10x for my specific use case" - Here is how](https://www.youtube.com/watch?v=Q9zv369Ggfk)

## ChatGLM
- [chaglm2 loRA finetuning](https://github.com/THUDM/ChatGLM2-6B/issues/114)

## LLaMA
- [用Lora和deepspeed微调LLaMA2-Chat](https://github.com/git-cloner/llama2-lora-fine-tuning)
- LLaMA-Efficient-Tuning https://github.com/hiyouga/LLaMA-Efficient-Tuning
   - Easy-to-use fine-tuning framework using PEFT (PT+SFT+RLHF with QLoRA) (LLaMA-2, BLOOM, Falcon, Baichuan, Qwen)

# 参考
- [**A High-level Overview of Large Language Models**](https://www.borealisai.com/research-blogs/a-high-level-overview-of-large-language-models/)
- [Prompt learning系列之prompt engineering(三) 连续型prompt自动构建](https://mp.weixin.qq.com/s?__biz=Mzk0NzMwNjU5Nw==&mid=2247483975&idx=1&sn=6d385e2312d7eb9dbebb95f3f0e43a6e&chksm=c379ab4df40e225b835c853a1e026a9fc709715fa7aa554f2cfcfb2d75fc491381d5c0ea1183&scene=21#wechat_redirect)
- [Prompt learning系列之prompt engineering(四) Multi prompt learning](https://mp.weixin.qq.com/s?__biz=Mzk0NzMwNjU5Nw==&mid=2247484033&idx=1&sn=217bbb1fbc7763cd2e22282a2a8a0737&chksm=c379ab8bf40e229d79870171315c489dac2163a1bfdca58582853da50595105ab24dc50f527d&scene=21#wechat_redirect)
- [Prompt learning系列之训练策略篇](https://mp.weixin.qq.com/s?__biz=Mzk0NzMwNjU5Nw==&mid=2247484043&idx=1&sn=2155c5b3571ccea28e7a4c048770d398&chksm=c379ab81f40e229738b42dd323da9132c646f6ba70958bdae2a8f6c74f3ac95758124b20b6e1&scene=21#wechat_redirect)
