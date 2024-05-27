# WhatsApp

这篇笔记展示了如何使用 WhatsApp 聊天加载器。这个类有助于将导出的 WhatsApp 对话映射到 LangChain 聊天消息。

这个过程有三个步骤：

1. 将聊天对话导出到计算机上

2. 使用文件路径指向的 json 文件或 JSON 文件目录创建 `WhatsAppChatLoader`

3. 调用 `loader.load()`（或 `loader.lazy_load()`）来执行转换。

## 1. 创建消息转储

要导出 WhatsApp 对话，完成以下步骤：

1. 打开目标对话

2. 点击右上角的三个点，选择“更多”

3. 然后选择“导出聊天”，并选择“不带媒体”

下面是每个对话的数据格式示例：

```python
%%writefile whatsapp_chat.txt
[8/15/23, 9:12:33 AM] Dr. Feather: ‎Messages and calls are end-to-end encrypted. No one outside of this chat, not even WhatsApp, can read or listen to them.
[8/15/23, 9:12:43 AM] Dr. Feather: I spotted a rare Hyacinth Macaw yesterday in the Amazon Rainforest. Such a magnificent creature!
‎[8/15/23, 9:12:48 AM] Dr. Feather: ‎image omitted
[8/15/23, 9:13:15 AM] Jungle Jane: That's stunning! Were you able to observe its behavior?
‎[8/15/23, 9:13:23 AM] Dr. Feather: ‎image omitted
[8/15/23, 9:14:02 AM] Dr. Feather: Yes, it seemed quite social with other macaws. They're known for their playful nature.
[8/15/23, 9:14:15 AM] Jungle Jane: How's the research going on parrot communication?
‎[8/15/23, 9:14:30 AM] Dr. Feather: ‎image omitted
[8/15/23, 9:14:50 AM] Dr. Feather: It's progressing well. We're learning so much about how they use sound and color to communicate.
[8/15/23, 9:15:10 AM] Jungle Jane: That's fascinating! Can't wait to read your paper on it.
[8/15/23, 9:15:20 AM] Dr. Feather: Thank you! I'll send you a draft soon.
[8/15/23, 9:25:16 PM] Jungle Jane: Looking forward to it! Keep up the great work.
```

```output
Writing whatsapp_chat.txt
```

## 2. 创建聊天加载器

WhatsAppChatLoader 接受生成的 zip 文件、解压后的目录，或其中任何聊天 `.txt` 文件的路径。同时提供您希望在微调时扮演“AI”角色的用户名。

```python
from langchain_community.chat_loaders.whatsapp import WhatsAppChatLoader
```

```python
loader = WhatsAppChatLoader(
    path="./whatsapp_chat.txt",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回当前存储加载对话每条消息的“ChatSessions”列表。

```python
from typing import List
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession
raw_messages = loader.lazy_load()
# 将同一发送者连续的消息合并为一条消息
merged_messages = merge_chat_runs(raw_messages)
# 将消息从“Dr. Feather”转换为 AI 消息
messages: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="Dr. Feather")
)
```

```output
[{'messages': [AIMessage(content='I spotted a rare Hyacinth Macaw yesterday in the Amazon Rainforest. Such a magnificent creature!', additional_kwargs={'sender': 'Dr. Feather', 'events': [{'message_time': '8/15/23, 9:12:43 AM'}]}, example=False),
   HumanMessage(content="That's stunning! Were you able to observe its behavior?", additional_kwargs={'sender': 'Jungle Jane', 'events': [{'message_time': '8/15/23, 9:13:15 AM'}]}, example=False),
   AIMessage(content="Yes, it seemed quite social with other macaws. They're known for their playful nature.", additional_kwargs={'sender': 'Dr. Feather', 'events': [{'message_time': '8/15/23, 9:14:02 AM'}]}, example=False),
   HumanMessage(content="How's the research going on parrot communication?", additional_kwargs={'sender': 'Jungle Jane', 'events': [{'message_time': '8/15/23, 9:14:15 AM'}]}, example=False),
   AIMessage(content="It's progressing well. We're learning so much about how they use sound and color to communicate.", additional_kwargs={'sender': 'Dr. Feather', 'events': [{'message_time': '8/15/23, 9:14:50 AM'}]}, example=False),
   HumanMessage(content="That's fascinating! Can't wait to read your paper on it.", additional_kwargs={'sender': 'Jungle Jane', 'events': [{'message_time': '8/15/23, 9:15:10 AM'}]}, example=False),
   AIMessage(content="Thank you! I'll send you a draft soon.", additional_kwargs={'sender': 'Dr. Feather', 'events': [{'message_time': '8/15/23, 9:15:20 AM'}]}, example=False),
   HumanMessage(content='Looking forward to it! Keep up the great work.', additional_kwargs={'sender': 'Jungle Jane', 'events': [{'message_time': '8/15/23, 9:25:16 PM'}]}, example=False)]}]
```

### 下一步

然后，您可以根据需要使用这些消息，例如微调模型、few-shot 示例选择，或直接对下一条消息进行预测。

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()
for chunk in llm.stream(messages[0]["messages"]):
    print(chunk.content, end="", flush=True)
```

感谢您的鼓励！我会尽力继续学习并分享关于鹦鹉交流的有趣见解。