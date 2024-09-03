# 数据准备
CONLL2003原始数据集
```
{
  'id': '0', 
  'tokens': ['EU', 'rejects', 'German', 'call', 'to', 'boycott', 'British', 'lamb', '.'],
  'ner_tags': ["B-ORGANIZATION", "O", "B-MISCELLANEOUS", "O", "O", "O", "B-MISCELLANEOUS", "O", "O"],
  'sentence': 'EU rejects German call to boycott British lamb.'
}
```
# 模型
```
openai_query_params = {
  "model": "text-davinci-003",
  "temperature": 0,
  "max_tokens": 1024
}
```

# 提示词
```
In the sentence below, give me the list of:
- organization named entity
- location named entity
- person named entity
- miscellaneous named entity.
Format the output in json with the following keys:
- ORGANIZATION for organization named entity
- LOCATION for location named entity
- PERSON for person named entity
- MISCELLANEOUS for miscellaneous named entity.
Sentence below:
```

# 测试结果
```
>>> test_sentence = (
    "Elon Musk is the CEO of Tesla and SpaceX. He was born in South Africa and now lives in the"
    " USA. He is one of the founders of OpenAI."
)
>>> print(ask_openai(base_prompt + test_sentence))
{
  "ORGANIZATION": ["Tesla", "SpaceX", "OpenAI"],
  "LOCATION": ["South Africa", "USA"],
  "PERSON": ["Elon Musk"],
  "MISCELLANEOUS": []
}
```

# 多角色提示词，提升抽取效果
```
SYSTEM_PROMPT = "You are a smart and intelligent Named Entity Recognition (NER) system. I will provide you the definition of the entities you need to extract, the sentence from where your extract the entities and the output format with examples."

USER_PROMPT_1 = "Are you clear about your role?"

ASSISTANT_PROMPT_1 = "Sure, I'm ready to help you with your NER task. Please provide me with the necessary information to get started."

GUIDELINES_PROMPT = (
    "Entity Definition:\n"
    "1. PERSON: Short name or full name of a person from any geographic regions.\n"
    "2. DATE: Any format of dates. Dates can also be in natural language.\n"
    "3. LOC: Name of any geographic location, like cities, countries, continents, districts etc.\n"
    "\n"
    "Output Format:\n"
    "{{'PERSON': [list of entities present], 'DATE': [list of entities present], 'LOC': [list of entities present]}}\n"
    "If no entities are presented in any categories keep it None\n"
    "\n"
    "Examples:\n"
    "\n"
    "1. Sentence: Mr. Jacob lives in Madrid since 12th January 2015.\n"
    "Output: {{'PERSON': ['Mr. Jacob'], 'DATE': ['12th January 2015'], 'LOC': ['Madrid']}}\n"
    "\n"
    "2. Sentence: Mr. Rajeev Mishra and Sunita Roy are friends and they meet each other on 24/03/1998.\n"
    "Output: {{'PERSON': ['Mr. Rajeev Mishra', 'Sunita Roy'], 'DATE': ['24/03/1998'], 'LOC': ['None']}}\n"
    "\n"
    "3. Sentence: {}\n"
    "Output: "
)
```
执行代码：
```python
import openai

openai.api_key = "OPENAI_API_KEY"

def openai_chat_completion_response(final_prompt):
  response = openai.ChatCompletion.create(
              model="gpt-3.5-turbo",
              messages=[
                    {"role": "system", "content": SYSTEM_PROMPT},
                    {"role": "user", "content": USER_PROMPT_1},
                    {"role": "assistant", "content": ASSISTANT_PROMPT_1},
                    {"role": "user", "content": final_prompt}
                ]
            )

  return response['choices'][0]['message']['content'].strip(" \n")
```

# 参考
- [Using ChatGPT to Pre-annotate Named Entities Recognition Labeling Tasks](https://kili-technology.com/data-labeling/machine-learning/using-chatgpt-to-pre-annotate-named-entities-recognition-labeling-tasks)
- [Zero-Shot Named Entity Recognition using OpenAI ChatGPT API](https://sourajit16-02-93.medium.com/zero-shot-named-entity-recognition-using-openai-chatgpt-api-46738191f375)
- [GPT-NER: Named Entity Recognition via Large Language Models](https://arxiv.org/pdf/2304.10428.pdf)
- [【论文精读】GPT-NER: Named Entity Recognition via Large Language Models](https://blog.csdn.net/HERODING23/article/details/130476395)
- https://github.com/ShuheWang1998/GPT-NER
- [How far is Language Model from 100% Few-shot Named Entity Recognition in Medical Domain](https://arxiv.org/pdf/2307.00186.pdf) 微调后 f1 > 0.9 低资源解决方案
- [PromptNER : Prompting For Named Entity Recognition](https://arxiv.org/pdf/2305.15444.pdf)
- [Automated Concatenation of Embeddings for Structured Prediction](https://arxiv.org/pdf/2010.05006.pdf)
- [微调BaiChuan13B来做命名实体识别](https://zhuanlan.zhihu.com/p/645339671) 带训练微调步骤 f1 = 0.87
