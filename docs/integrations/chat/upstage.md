---

sidebar_label: Upstage

---

# 聊天Upstage

本文介绍如何开始使用Upstage聊天模型。

## 安装

安装 `langchain-upstage` 包。

```bash
pip install -U langchain-upstage
```

## 环境设置

确保设置以下环境变量：

- `UPSTAGE_API_KEY`: 您从[Upstage控制台](https://console.upstage.ai/)获取的Upstage API密钥。

## 使用

```python
import os
os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_upstage import ChatUpstage
chat = ChatUpstage()
```

```python
# 使用chat invoke
chat.invoke("你好，你好吗？")
```

```python
# 使用chat stream
for m in chat.stream("你好，你好吗？"):
    print(m)
```

## 链接

```python
# 使用chain
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "您是一个乐于助人的助手，可以将英语翻译成法语。"),
        ("human", "将这句话从英语翻译成法语。{english_text}."),
    ]
)
chain = prompt | chat
chain.invoke({"english_text": "你好，你好吗？"})
```