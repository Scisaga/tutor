- https://cloud.google.com/vertex-ai?hl=zh-cn
- [Vertex AI 使用入门](https://console.cloud.google.com/vertex-ai?hl=zh-cn)

# 文本数据

## 提示词设计

[Overview of text prompt design](https://cloud.google.com/vertex-ai/docs/generative-ai/text/text-overview)，示例：
```
Text:
Question:
Answer:
Categories:
Options:
```
示例2（关联一致性）：
```
Classify the sentiment of the following text as positive or negative.
Text: I love chocolate.
Sentiment:
```

## 分类
- [解读文本分类模型的预测结果](https://cloud.google.com/vertex-ai/docs/text-data/classification/interpret-results?hl=zh-cn)

## 总结
- [Design summarization prompts](https://cloud.google.com/vertex-ai/docs/generative-ai/text/summarization-prompts)

### 总结文本
```
Provide a summary for the following article:
...
Write an abstract of this article:
...
Write a creative title for the text.
...
```

### 总结对话
```
Summarize the following conversation.
A: ...
B: ...
```

## 提取
- [Design extraction prompts](https://cloud.google.com/vertex-ai/docs/generative-ai/text/extraction-prompts)
### 使用场景
The following are common use cases for extraction:
- Named entity recognition (NER): Extract named entities from text, including people, places, organizations, and dates.
- Relation extraction: Extract the relationships between entities in text, such as family relationships between people.
- Event extraction: Extract events from text, such as project milestones and product launches.
- Question answering: Extract information from text to answer a question.
### 模型
```
text-bison
```
### 示例
Prompt:
```
Extract the technical specifications from the text below in a JSON format. Valid fields are name, network, ram, processor, storage, and color.
Text: Google Pixel 7, 5G network, 8GB RAM, Tensor G2 processor, 128GB of storage, Lemongrass
JSON:
```
Response:
```
{
  "name": "Google Pixel 7",
  "network": "5G",
  "ram": "8GB",
  "processor": "Tensor G2",
  "storage": "128GB",
  "color": "Lemongrass"
}
```
## Vertex AI API
- [Quickstart using the Vertex AI API](https://cloud.google.com/vertex-ai/docs/generative-ai/start/quickstarts/api-quickstart) 文中给了提示词示例和参数示例，重点参考
- [从文本实体提取模型获取预测结果](https://cloud.google.com/vertex-ai/docs/text-data/entity-extraction/get-predictions?hl=zh-cn#deploy_a_model_to_an_endpoint) 支持批量预测
- [Google Vertex AI Client for Java](https://github.com/googleapis/google-cloud-java/tree/main/java-aiplatform)

# 价格
- [Vertex AI 价格](https://cloud.google.com/vertex-ai/pricing?hl=zh-cn)
