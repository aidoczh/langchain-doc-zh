# DeepInfra

[DeepInfra](https://deepinfra.com/?utm_source=langchain) 是一个无服务器推理服务，提供访问各种 [LLM 模型](https://deepinfra.com/models?utm_source=langchain) 和 [嵌入模型](https://deepinfra.com/models?type=embeddings&utm_source=langchain)。本笔记将介绍如何使用 LangChain 与 DeepInfra 一起使用语言模型。

## 设置环境 API 密钥

确保从 DeepInfra 获取您的 API 密钥。您需要[登录](https://deepinfra.com/login?from=%2Fdash) 并获取一个新的令牌。

您将获得 1 小时免费的无服务器 GPU 计算时间来测试不同的模型（参见[这里](https://github.com/deepinfra/deepctl#deepctl)）。

您可以使用 `deepctl auth token` 命令打印您的令牌。

```python
# 获取新令牌：https://deepinfra.com/login?from=%2Fdash
from getpass import getpass
DEEPINFRA_API_TOKEN = getpass()
```

```output
 ········
```

```python
import os
os.environ["DEEPINFRA_API_TOKEN"] = DEEPINFRA_API_TOKEN
```

## 创建 DeepInfra 实例

您还可以使用我们的开源 [deepctl 工具](https://github.com/deepinfra/deepctl#deepctl) 来管理您的模型部署。您可以在[这里](https://deepinfra.com/databricks/dolly-v2-12b#API)查看可用参数的列表。

```python
from langchain_community.llms import DeepInfra
llm = DeepInfra(model_id="meta-llama/Llama-2-70b-chat-hf")
llm.model_kwargs = {
    "temperature": 0.7,
    "repetition_penalty": 1.2,
    "max_new_tokens": 250,
    "top_p": 0.9,
}
```

```python
# 通过包装器直接运行推理
llm("Who let the dogs out?")
```

```output
'这是一个让许多人感到困惑的问题'
```

```python
# 运行流式推理
for chunk in llm.stream("Who let the dogs out?"):
    print(chunk)
```

```output
威尔·史密斯
。
```

## 创建提示模板

我们将为问题和答案创建一个提示模板。

```python
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: 让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain

```python
from langchain.chains import LLMChain
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain

提供一个问题并运行 LLMChain。

```python
question = "企鹅能到达北极吗？"
llm_chain.run(question)
```

```output
"企鹅生活在南极洲及其周围岛屿，这些地方位于地球的最南端。而北极位于地球的最北端，对企鹅来说要到达那里将是一段漫长的旅程。事实上，企鹅没有飞行或迁徙到如此遥远的距离的能力。所以，不，企鹅无法到达北极。"
```