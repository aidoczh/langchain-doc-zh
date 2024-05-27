# GooseAI

`GooseAI` 是一项完全托管的 NLP 服务，通过 API 提供。GooseAI 提供了访问[这些模型](https://goose.ai/docs/models)的途径。

本笔记将介绍如何使用 Langchain 与 [GooseAI](https://goose.ai/)。

## 安装 openai

使用 GooseAI API 需要安装 `openai` 包。可以通过 `pip install openai` 进行安装。

```python
%pip install --upgrade --quiet langchain-openai
```

## 导入模块

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import GooseAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥

确保从 GooseAI 获取您的 API 密钥。您将获得价值 10 美元的免费信用额度来测试不同的模型。

```python
from getpass import getpass
GOOSEAI_API_KEY = getpass()
```

```python
os.environ["GOOSEAI_API_KEY"] = GOOSEAI_API_KEY
```

## 创建 GooseAI 实例

您可以指定不同的参数，如模型名称、生成的最大标记数、温度等。

```python
llm = GooseAI()
```

## 创建一个提示模板

我们将为问题和答案创建一个提示模板。

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain

提供一个问题并运行 LLMChain。

```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"
llm_chain.run(question)
```