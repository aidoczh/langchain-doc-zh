# ForefrontAI

`Forefront` 平台让您能够微调并使用[开源大型语言模型](https://docs.forefront.ai/forefront/master/models)。

本笔记将介绍如何使用 Langchain 与 [ForefrontAI](https://www.forefront.ai/)。

## 引入模块

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import ForefrontAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥

确保从 ForefrontAI 获取您的 API 密钥。您将获得为期 5 天的免费试用期来测试不同的模型。

```python
# 获取新的令牌：https://docs.forefront.ai/forefront/api-reference/authentication
from getpass import getpass
FOREFRONTAI_API_KEY = getpass()
```

```python
os.environ["FOREFRONTAI_API_KEY"] = FOREFRONTAI_API_KEY
```

## 创建 ForefrontAI 实例

您可以指定不同的参数，如模型端点 URL、长度、温度等。您必须提供一个端点 URL。

```python
llm = ForefrontAI(endpoint_url="在此处填入您的端点 URL")
```

## 创建提示模板

我们将为问题和答案创建一个提示模板。

```python
template = """Question: {question}
Answer: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain

提供一个问题并运行 LLMChain。

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```