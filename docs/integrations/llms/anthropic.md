---

sidebar_label: 人类中心LLM

---

# 人类中心LLM

本示例介绍如何使用LangChain与`人类中心`模型进行交互。

注意：人类中心LLM仅支持传统的Claude 2模型。要使用最新的Claude 3模型，请改用[`ChatAnthropic`](/docs/integrations/chat/anthropic)。

## 安装

```python
%pip install -qU langchain-anthropic
```

## 环境设置

我们需要获取一个[人类中心](https://console.anthropic.com/settings/keys) API密钥，并设置`ANTHROPIC_API_KEY`环境变量：

```python
import os
from getpass import getpass
os.environ["ANTHROPIC_API_KEY"] = getpass()
```

## 使用

```python
from langchain_anthropic import AnthropicLLM
from langchain_core.prompts import PromptTemplate
template = """问题：{question}
答案：让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
model = AnthropicLLM(model="claude-2.1")
chain = prompt | model
chain.invoke({"question": "什么是LangChain?"})
```

```output
'\nLangChain是一个利用人工智能和机器学习提供语言翻译服务的去中心化区块链网络。'
```