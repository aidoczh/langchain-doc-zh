# DeepInfra

[DeepInfra](https://deepinfra.com/?utm_source=langchain) 是一项无服务器推理服务，提供对[各种LLM模型](https://deepinfra.com/models?utm_source=langchain)和[嵌入模型](https://deepinfra.com/models?type=embeddings&utm_source=langchain)的访问。本笔记介绍了如何在DeepInfra中使用LangChain进行聊天模型。

## 设置环境API密钥

确保从DeepInfra获取您的API密钥。您需要[登录](https://deepinfra.com/login?from=%2Fdash)并获取一个新的令牌。

您将获得1小时的免费无服务器GPU计算时间来测试不同的模型。（参见[这里](https://github.com/deepinfra/deepctl#deepctl)）

您可以使用 `deepctl auth token` 命令打印您的令牌

```python
# 获取一个新的令牌：https://deepinfra.com/login?from=%2Fdash
import os
from getpass import getpass
from langchain_community.chat_models import ChatDeepInfra
from langchain_core.messages import HumanMessage
DEEPINFRA_API_TOKEN = getpass()
# 或者将 deepinfra_api_token 参数传递给 ChatDeepInfra 构造函数
os.environ["DEEPINFRA_API_TOKEN"] = DEEPINFRA_API_TOKEN
chat = ChatDeepInfra(model="meta-llama/Llama-2-7b-chat-hf")
messages = [
    HumanMessage(
        content="将这句话从英语翻译成法语。我喜欢编程。"
    )
]
chat.invoke(messages)
```

## `ChatDeepInfra` 还支持异步和流式功能：

```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
```

```python
await chat.agenerate([messages])
```

```python
chat = ChatDeepInfra(
    streaming=True,
    verbose=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
chat.invoke(messages)
```