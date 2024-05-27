# Slack

这篇笔记展示了如何使用 Slack 聊天加载器。这个类有助于将导出的 Slack 对话映射到 LangChain 聊天消息。

该过程分为三步：

1. 按照[这里的说明](https://slack.com/help/articles/1500001548241-Request-to-export-all-conversations)导出所需的对话线程。

2. 创建 `SlackChatLoader`，并将文件路径指向 json 文件或 JSON 文件目录。

3. 调用 `loader.load()`（或 `loader.lazy_load()`）执行转换。可选择使用 `merge_chat_runs` 将同一发件人的消息按顺序合并，和/或使用 `map_ai_messages` 将指定发件人的消息转换为 "AIMessage" 类。

## 1. 创建消息转储

目前（2023/08/23），该加载器最好支持以 zip 目录形式的文件，这些文件是通过从 Slack 导出直接消息对话生成的格式。请查看 Slack 的最新说明。

我们在 LangChain 存储库中有一个示例。

```python
import requests
permalink = "https://raw.githubusercontent.com/langchain-ai/langchain/342087bdfa3ac31d622385d0f2d09cf5e06c8db3/libs/langchain/tests/integration_tests/examples/slack_export.zip"
response = requests.get(permalink)
with open("slack_dump.zip", "wb") as f:
    f.write(response.content)
```

## 2. 创建聊天加载器

向加载器提供指向 zip 目录的文件路径。您还可以选择指定映射到 AI 消息的用户 ID，并配置是否合并消息运行。

```python
from langchain_community.chat_loaders.slack import SlackChatLoader
```

```python
loader = SlackChatLoader(
    path="slack_dump.zip",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回一个“ChatSessions”列表，目前每个加载的对话只包含一系列消息。

```python
from typing import List
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession
raw_messages = loader.lazy_load()
# 将同一发件人连续的消息合并为一条消息
merged_messages = merge_chat_runs(raw_messages)
# 将消息从 "U0500003428" 转换为 AI 消息
messages: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="U0500003428")
)
```

### 下一步

然后，您可以根据需要使用这些消息，例如微调模型、选择少量示例，或直接对下一条消息进行预测。

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()
for chunk in llm.stream(messages[1]["messages"]):
    print(chunk.content, end="", flush=True)
```

```output
嗨，
希望一切都好。我想联系你，问问你下周是否有空喝杯咖啡。我很想聊聊，听听你最近的生活如何。如果你有兴趣，告诉我，我们可以找个适合我们双方的时间。
期待你的回复！
祝好，[你的名字]
```