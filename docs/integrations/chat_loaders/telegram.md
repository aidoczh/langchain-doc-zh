# 电报

这篇笔记展示了如何使用电报聊天加载器。这个类有助于将导出的电报对话映射到 LangChain 聊天消息。

这个过程有三个步骤：

1. 通过将电报应用中的聊天复制并粘贴到本地计算机上的文件中，导出聊天 .txt 文件

2. 使用文件路径指向的 json 文件或 JSON 文件目录创建 `TelegramChatLoader`

3. 调用 `loader.load()`（或 `loader.lazy_load()`）执行转换。可选择使用 `merge_chat_runs` 将来自同一发送者的消息合并在一起，和/或使用 `map_ai_messages` 将来自指定发送者的消息转换为 "AIMessage" 类。

## 1. 创建消息转储

目前（2023/08/23），该加载器最好支持由[电报桌面应用](https://desktop.telegram.org/)导出的格式的 json 文件。

**重要提示：** 有一些 '轻量级' 版本的电报，比如 "Telegram for MacOS"，它们缺乏导出功能。请确保您使用正确的应用程序导出文件。

进行导出：

1. 下载并打开电报桌面应用

2. 选择一个对话

3. 转到对话设置（目前在右上角的三个点）

4. 点击 "导出聊天记录"

5. 取消选择照片和其他媒体。选择 "机器可读 JSON" 格式进行导出。

以下是一个示例：

```python
%%writefile telegram_conversation.json
{
 "name": "Jiminy",
 "type": "personal_chat",
 "id": 5965280513,
 "messages": [
  {
   "id": 1,
   "type": "message",
   "date": "2023-08-23T13:11:23",
   "date_unixtime": "1692821483",
   "from": "Jiminy Cricket",
   "from_id": "user123450513",
   "text": "You better trust your conscience",
   "text_entities": [
    {
     "type": "plain",
     "text": "You better trust your conscience"
    }
   ]
  },
  {
   "id": 2,
   "type": "message",
   "date": "2023-08-23T13:13:20",
   "date_unixtime": "1692821600",
   "from": "Batman & Robin",
   "from_id": "user6565661032",
   "text": "What did you just say?",
   "text_entities": [
    {
     "type": "plain",
     "text": "What did you just say?"
    }
   ]
  }
 ]
}
```

```output
Overwriting telegram_conversation.json
```

## 2. 创建聊天加载器

所需的只是文件路径。您还可以选择指定映射到 AI 消息的用户名，以及配置是否合并消息运行。

```python
from langchain_community.chat_loaders.telegram import TelegramChatLoader
```

```python
loader = TelegramChatLoader(
    path="./telegram_conversation.json",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回一个 "ChatSessions" 列表，目前只包含每个加载的对话的消息列表。

```python
from typing import List
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession
raw_messages = loader.lazy_load()
# 将连续的来自同一发送者的消息合并为一条消息
merged_messages = merge_chat_runs(raw_messages)
# 将来自 "Jiminy Cricket" 的消息转换为 AI 消息
messages: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="Jiminy Cricket")
)
```

### 下一步

然后，您可以根据需要使用这些消息，例如微调模型、选择少量示例，或直接为下一条消息做出预测。

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()
for chunk in llm.stream(messages[0]["messages"]):
    print(chunk.content, end="", flush=True)
```

```output
我说过，“你最好相信你的良心。”
```