ChatGPT 能够以类似人类的答案来响应各种基于文本的查询。 凭借其庞大的训练数据，ChatGPT 可以提供适合上下文的答案，使其成为语言翻译、客户服务和内容创建的有效工具，ChatGPT主要应用领域包括：
- **问答**：ChatGPT 可用于根据其训练数据回答问题。 它可以生成各种问题的
- **文本生成**：ChatGPT 可用于生成文本，例如故事、诗歌和新闻文章。 它还可用于为聊天机器人和虚拟助理生成文本。
- **语言翻译**：ChatGPT 可以进行微调来执行语言翻译任务，将文本从一种语言翻译成另一种语言。
- **文本摘要**：ChatGPT 可用于将长文本摘要为更短、更简洁的摘要。
- **聊天机器人**：ChatGPT 可用作对话式 AI 来开发用于客户服务、销售和其他应用程序的聊天机器人。
- **情感分析**：ChatGPT 可以进行微调以执行情感分析，其中涉及确定一段文本中表达的情绪（积极、消极、中立）。
- **命名实体识别**：ChatGPT 可以进行微调以执行命名实体识别，其中涉及识别文本中的命名实体（例如人物、地点和组织）。

# 常用地址
- https://openai.com/
- https://openai.com/blog/chatgpt
- https://chat.openai.com/
- [OpenAI API and other LLM APIs response time tracker](https://gptforwork.com/tools/openai-api-and-other-llm-apis-response-time-tracker)
- [ChatGPT/GPT-3/GPT-4 models guide](https://gptforwork.com/guides/openai-gpt3-models)
- [OpenAI GPT prompt generator](https://gptforwork.com/tools/prompt-generator)
- [OpenAI API Pricing calculator](https://gptforwork.com/tools/openai-chatgpt-api-pricing-calculator)

## Docs & API
- https://platform.openai.com/examples
- https://platform.openai.com/docs/introduction

## 短信验证码服务
- https://sms-activate.org/getNumber

# 基本概念

- Token：按照一定的规则进行拆分，比如hamburger，被认为是3个token，分别是：ham、bur、ger；另外一个汉字是2个token。通过这个网址来测试内容有多少个token：https://platform.openai.com/tokenizer
- Model：具有推理能力的模型，不同模型的能力差异巨大，API调用费用也可以有百倍以上的差别
- Prompt：提示词，提问的内容。构建有效的提示词可以让模型给出更精确的结果，构建提示词本身都逐渐形成了一个专门的领域。
- Text completion：文本补全，让模型根据提示词内容返回结果
- Temperature：温度，0-1，0：风格更固定，1：风格更加多样化
- Fine tune：模型精调
- Retrieve：检索，找回

## 模型对比

常用模型列表：

- GPT-4：截至目前位置最强大的AI模型
- GPT-3.5：免费的最新的模型，训练数据时间截至21年9月，可以满足绝大多数场景
- DALL·E (Beta)：用于生成和编辑图像的模型
- Whisper (Beta)：用于将音频转换成文本的模型
- Embeddings：将文本转换成数字的模型
- Moderation：监测文本是否敏感和安全的模型（比如是否包含：暴力、黄色、仇恨、自我伤害等）

Davinci model:
- GPT-Davinci模型是OpenAI开发的GPT语言模型最强大的版本之一。 它是一个大型语言模型，于 2021 年 6 月发布，作为 GPT-3 模型系列的一部分。 GPT-Davinci 模型拥有 1750 亿个参数。
- 为各种任务生成高质量的自然语言响应，包括语言翻译、摘要、问答等。 它能够理解语言的上下文和细微差别，并能够生成连贯且有意义的响应。
- GPT-Davinci 模型的关键特征之一是其执行零样本学习的能力，这意味着它可以执行尚未明确训练的任务。 这是通过其大量参数和所经历的广泛预训练来实现的，这使得它能够从大量语言数据中学习和概括。

Turbo model:
- 60亿参数，精确度较低，生成长文本能力有限，主要用于对话

### GPT-3.5 与 GPT-4 的区别

- **模型规模**：GPT4和GPT3.5的主要区别在于模型的规模。GPT4预计将拥有超过100万亿个参数（1e+15），而GPT3只有1750亿个参数（1.75e+11）。这意味着GPT4可以处理更多的数据，生成更长、更复杂、更连贯、更准确、更多样化和更有创造力的文本。
- **模型能力**：由于模型规模的提升，GPT4也展现出了比GPT3.5更强大的能力。例如，在各种专业和学术考试中，如SAT、LSAT、GRE等，GPT4都表现出了与人类水平相当或超越的性能；而在日常对话中，也能够与人类进行流畅、自然、合理、有趣且富有逻辑性的交流。
- **模型输入**：GPT4是一个多模态（multimodal）模型，即它可以接受图像和文本作为输入，并输出文本；而GPT3.5只能接受文本作为输入，并输出文本。这使得GPT4可以处理更复杂且具有视觉信息的任务，如图像描述、图像问答、图像到文本等。
- **模型训练**：由于数据量和计算资源的限制，目前没有公开发布完整版的GPT4或者其训练代码；而OpenAI已经公开了部分版本（如Davinci）以及其API接口供用户使用或测试。

# 使用配额 & 收费标准

## GPT-4网页版
- http://chat.openai.com/ ，ChatGPT Plus subscribers，50条消息每3小时。（2023/07/06）

## API调用
- 充值麻烦，需要国外银行卡
- 使用计费说明：按回答内容计费，API调用计费方式：总消耗=提问消耗+答案消耗
- 1K tokens 大约750个英文单词，375个汉字
- 免费用户：200请求每天，3请求每分钟，15wToken每分钟，免费账户有$5的额度，注册后90天内有效
- GPT-4 8K context：Prompt $0.03 / 1K tokens，Completion $0.06 / 1K tokens
- GPT-4 32K context：Prompt $0.06 / 1K tokens，Completion $0.12 / 1K tokens
- gpt-3.5-turbo：$0.002 / 1K tokens
- [OpenAI Rate limits](https://platform.openai.com/docs/guides/rate-limits/overview) | [问一下现在OpenAI的计费是怎么样的？](https://github.com/LlmKira/Openaibot/issues/19)

## 模型微调（Fine-tuning models）

<table><tbody><tr><td>模型</td><td>训练费用 / 1K token</td><td>使用费用 / 1K token</td></tr><tr><td>Ada</td><td>$0.0004</td><td>$0.0016</td></tr><tr><td>Babbage</td><td>$0.0006</td><td>$0.0024</td></tr><tr><td>Curie</td><td>$0.0030</td><td>$0.0120</td></tr><tr><td>Davinci</td><td>$0.0300</td><td>$0.1200</td></tr></tbody></table>

# 提示词

基本指南：
1. 提供更多的细节
2. 要求模型进行角色扮演
3. 使用分隔符清楚的指示输入的不同部分
4. 指定完成任务的具体步骤
5. 提供例子
6. 指定输出的长度

三层提示词结构：
1. 先说明目的，最总交付的内容时什么
2. 输入资料，说明任务、北京、相关的人物、事件、时间、地点和物品
3. 设定输出格式：输出几个、特殊格式（问题-答案）、使用的语气风格、是否使用表格、JSON或Markdown

## 高级技巧

### 上下文学习
### 思维链
### Prompt 模板
### 对抗性提示
### 自动Prompt工程

## 累计推理 CR
- [Cumulative Reasoning With Large Language Models](https://arxiv.org/abs/2308.04371v1)
   - 使用累积和迭代的语言模型来模拟人类思考过程。通过将任务分解成更小的部分,我们简化了解决问题的流程,使其更加可管理且更有效。

## 参考

- [Awesome ChatGPT Prompts](https://github.com/f/awesome-chatgpt-prompts") | https://prompts.chat/
- [AI Short](https://www.aishort.top/?tags=article)
- [面向开发者的 LLM 入门课程](https://github.com/datawhalechina/prompt-engineering-for-developers)
- [ChatGPT 提示語說明書：通用三層結構與 9 個技巧提高 AI 生產力](https://www.playpcesor.com/2023/04/chatgpt-9-ai.html)

# 插件/工具链

- [Custom instructions for ChatGPT](https://help.openai.com/en/articles/8096356-custom-instructions-for-chatgpt") 通过自定义提示词，可以让ChatGPT的返回结果更符合预期
- [如何让 ChatGPT 更懂你？新功能 Custom Instructions 尝试](https://sspai.com/post/81470) 使用自定义提示词更有效的生成Python代码
- [Prompt Engineering For Generative AI Gets Pumped Up Via Release Of Persistent Context Capabilities, Showcased By OpenAI’s New Custom Instructions ChatGPT Feature](https://www.forbes.com/sites/lanceeliot/2023/07/23/prompt-engineering-for-generative-ai-gets-pumped-up-via-release-of-persistent-context-capabilities-showcased-by-openais-new-custom-instructions-chatgpt-feature/?sh=47a783582020)
- https://gptstore.ai/ ChatGPT插件

# 模型微调

- [ChatGPT中的fine-tuning微调是如何进行实践的](https://juejin.cn/post/7218001191703068729)

# 常见问题

- [ChatGPT 的 4000 个 token 上下文不够用怎么办？](https://zhuanlan.zhihu.com/p/616860260)
- [如何进入开发者模式？](https://www.jailbreakchat.com/)

# 其他

- [OpenAI的实时服务状态](https://status.openai.com/)

# 参考

- [How to Use ChatGPT & OpenAI Without a Phone Number](https://www.wikihow.com/Chatgpt-Without-Phone-Number)
- [国内开通Chat GPT Plus保姆级教程](https://chatgpt-plus.github.io/chatgpt-plus/)
- [GPT best practices](https://platform.openai.com/docs/guides/gpt-best-practices)
- [完全免费白嫖 GPT-4 的终极方案！](https://juejin.cn/post/7241790368949190693) 想法很好
- [奥特曼不止ChatGPT](https://zhuanlan.zhihu.com/p/607713738)
- [The GDELT Project](https://blog.gdeltproject.org/) 不同提示词在ChatGPT/Bard的输出效果对比
- [是什么让CHATGPT变得如此聪明？思维链？涌现？](https://librariestoner.com/archives/chatcot)
- [Overview of LLM landscape: Usage, latency, pricing and more.](https://medium.com/@babblebots.ai/overview-of-llm-landscape-usage-latency-pricing-and-more-8775ad900590)
- [Battle of the LLM Giants: Google PaLM 2 vs OpenAI GPT-3.5](https://towardsdatascience.com/battle-of-the-llm-giants-google-palm-2-vs-openai-gpt-3-5-798802ddb53c)
- [Uncovering the power of top-notch LLMs](https://dataconomy.com/2023/07/18/best-large-language-models-llms/)
- [12 Best Large Language Models (LLMs) in 2023](https://beebom.com/best-large-language-models-llms/)
