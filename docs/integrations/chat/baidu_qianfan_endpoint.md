---

sidebar_label: 百度千帆

---

# 千帆聊天端点

百度AI云千帆平台是为企业开发者提供的一站式大型模型开发和服务运营平台。千帆不仅提供文心译言（ERNIE-Bot）模型和第三方开源模型，还提供各种人工智能开发工具和完整的开发环境，方便客户轻松使用和开发大型模型应用。

基本上，这些模型被分为以下类型：

- 嵌入（Embedding）

- 聊天（Chat）

- 完成（Completion）

在本笔记本中，我们将介绍如何在 [千帆](https://cloud.baidu.com/doc/WENXINWORKSHOP/index.html) 中主要使用 `Chat` 对应的 `langchain/chat_models` 包来使用 langchain 中的 langchain。

## API初始化

要使用基于百度千帆的LLM服务，您必须初始化这些参数：

您可以选择在环境变量中初始化AK、SK，也可以初始化参数：

```base
export QIANFAN_AK=XXX
export QIANFAN_SK=XXX
```

## 当前支持的模型：

- ERNIE-Bot-turbo（默认模型）

- ERNIE-Bot

- BLOOMZ-7B

- Llama-2-7b-chat

- Llama-2-13b-chat

- Llama-2-70b-chat

- Qianfan-BLOOMZ-7B-compressed

- Qianfan-Chinese-Llama-2-7B

- ChatGLM2-6B-32K

- AquilaChat-7B

## 设置

```python
"""用于基本初始化和调用"""
import os
from langchain_community.chat_models import QianfanChatEndpoint
from langchain_core.language_models.chat_models import HumanMessage
os.environ["QIANFAN_AK"] = "Your_api_key"
os.environ["QIANFAN_SK"] = "You_secret_Key"
```

## 用法

```python
chat = QianfanChatEndpoint(streaming=True)
messages = [HumanMessage(content="Hello")]
chat.invoke(messages)
```

```output
AIMessage(content='您好！请问您需要什么帮助？我将尽力回答您的问题。')
```

```python
await chat.ainvoke(messages)
```

```output
AIMessage(content='您好！有什么我可以帮助您的吗？')
```

```python
chat.batch([messages])
```

```output
[AIMessage(content='您好！有什么我可以帮助您的吗？')]
```

### 流式处理

```python
try:
    for chunk in chat.stream(messages):
        print(chunk.content, end="", flush=True)
except TypeError as e:
    print("")
```

```output
您好！有什么我可以帮助您的吗？
```

## 在千帆中使用不同的模型

默认模型是ERNIE-Bot-turbo，在您想要基于Ernie Bot或第三方开源模型部署自己的模型时，您可以按照以下步骤进行操作：

1. （可选，如果模型已包含在默认模型中，则跳过）在千帆控制台中部署您的模型，获取您自己定制的部署端点。

2. 在初始化中设置名为 `endpoint` 的字段：

```python
chatBot = QianfanChatEndpoint(
    streaming=True,
    model="ERNIE-Bot",
)
messages = [HumanMessage(content="Hello")]
chatBot.invoke(messages)
```

```output
AIMessage(content='Hello，可以回答问题了，我会竭尽全力为您解答，请问有什么问题吗？')
```

## 模型参数：

目前，只有 `ERNIE-Bot` 和 `ERNIE-Bot-turbo` 支持以下模型参数，我们将来可能会支持更多模型。

- 温度（temperature）

- top_p

- 惩罚分数（penalty_score）

```python
chat.invoke(
    [HumanMessage(content="Hello")],
    **{"top_p": 0.4, "temperature": 0.1, "penalty_score": 1},
)
```

```output
AIMessage(content='您好！有什么我可以帮助您的吗？')
```