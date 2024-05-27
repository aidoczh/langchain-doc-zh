# NLP Cloud

[NLP Cloud](https://nlpcloud.io) 提供高性能的预训练模型或自定义模型，用于命名实体识别（NER）、情感分析、分类、摘要、改写、语法和拼写纠正、关键词和关键短语提取、聊天机器人、产品描述和广告生成、意图分类、文本生成、图像生成、博客文章生成、代码生成、问答、自动语音识别、机器翻译、语言检测、语义搜索、语义相似度、分词、词性标注、嵌入和依存句法分析。它已经准备好投入生产，并通过 REST API 提供服务。

以下示例介绍了如何使用 LangChain 与 `NLP Cloud` [模型](https://docs.nlpcloud.com/#models) 进行交互。

```python
%pip install --upgrade --quiet  nlpcloud
```

```python
# 获取令牌：https://docs.nlpcloud.com/#authentication
from getpass import getpass
NLPCLOUD_API_KEY = getpass()
```

```output
 ········
```

```python
import os
os.environ["NLPCLOUD_API_KEY"] = NLPCLOUD_API_KEY
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import NLPCloud
from langchain_core.prompts import PromptTemplate
```

```python
template = """问题: {question}
回答: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = NLPCloud()
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生的那一年，哪支 NFL 球队赢得了超级碗？"
llm_chain.run(question)
```

```output
'贾斯汀·比伯出生于1994年，所以那一年赢得超级碗的球队是旧金山49人队。'
```