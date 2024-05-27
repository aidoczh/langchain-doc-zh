---

sidebar_label: 登月计划

---

# 登月聊天

[Moonshot](https://platform.moonshot.cn/) 是一家中国初创公司，为企业和个人提供 LLM 服务。

本示例介绍如何使用 LangChain 与 Moonshot 聊天推理进行交互。

```python
import os
# 从 https://platform.moonshot.cn/console/api-keys 生成你的 API 密钥
os.environ["MOONSHOT_API_KEY"] = "MOONSHOT_API_KEY"
```

```python
from langchain_community.chat_models.moonshot import MoonshotChat
from langchain_core.messages import HumanMessage, SystemMessage
```

```python
chat = MoonshotChat()
# 或者使用特定的模型
# 可用模型：https://platform.moonshot.cn/docs
# chat = MoonshotChat(model="moonshot-v1-128k")
```

```python
messages = [
    SystemMessage(
        content="你是一个有帮助的助手，可以将英语翻译成法语。"
    ),
    HumanMessage(
        content="将这个句子从英语翻译成法语。我喜欢编程。"
    ),
]
chat.invoke(messages)
```