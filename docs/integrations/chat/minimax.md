---

sidebar_label: MiniMax

---

# MiniMaxChat

[Minimax](https://api.minimax.chat) 是一家中国初创公司，为企业和个人提供 LLM 服务。

这个示例演示了如何使用 LangChain 与 MiniMax 推理聊天进行交互。

```python
import os
os.environ["MINIMAX_GROUP_ID"] = "MINIMAX_GROUP_ID"
os.environ["MINIMAX_API_KEY"] = "MINIMAX_API_KEY"
```

```python
from langchain_community.chat_models import MiniMaxChat
from langchain_core.messages import HumanMessage
```

```python
chat = MiniMaxChat()
```

```python
chat(
    [
        HumanMessage(
            content="Translate this sentence from English to French. I love programming."
        )
    ]
)
```