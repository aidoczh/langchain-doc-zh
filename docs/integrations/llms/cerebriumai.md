# CerebriumAI

`Cerebrium` 是 AWS Sagemaker 的替代品。它还提供了对[多个 LLM 模型的 API 访问](https://docs.cerebrium.ai/cerebrium/prebuilt-models/deployment)。

本文档介绍如何使用 [CerebriumAI](https://docs.cerebrium.ai/introduction) 使用 Langchain。

## 安装 cerebrium

使用 `pip3 install cerebrium` 命令安装 `cerebrium` 包以使用 `CerebriumAI` API。

```python
# 安装包
!pip3 install cerebrium
```

## 导入模块

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import CerebriumAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥

确保从 CerebriumAI 获取您的 API 密钥。请参阅[这里](https://dashboard.cerebrium.ai/login)。您将获得 1 小时的免费 GPU 计算时间来测试不同的模型。

```python
os.environ["CEREBRIUMAI_API_KEY"] = "YOUR_KEY_HERE"
```

## 创建 CerebriumAI 实例

您可以指定不同的参数，如模型端点 URL、最大长度、温度等。您必须提供一个端点 URL。

```python
llm = CerebriumAI(endpoint_url="YOUR ENDPOINT URL HERE")
```

## 创建一个 Prompt 模板

我们将为问题和答案创建一个 Prompt 模板。

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
question = "Justin Beiber 出生年份的时候，哪个 NFL 球队赢得了超级碗？"
llm_chain.run(question)
```