# 微信

目前还没有直接导出个人微信消息的简单方法。但是，如果你只需要少于几百条消息用于模型微调或少量示例，本文将展示如何创建自己的聊天加载器，将复制粘贴的微信消息转换为 LangChain 消息列表。

> 受 https://python.langchain.com/docs/integrations/chat_loaders/discord 启发

该过程分为五个步骤：

1. 在微信桌面应用中打开你的聊天记录。通过鼠标拖动或右键选择需要的消息。由于限制，一次最多只能选择100条消息。使用 `CMD`/`Ctrl` + `C` 进行复制。

2. 将选择的消息粘贴到本地计算机上的一个文件中，创建聊天记录的 .txt 文件。

3. 将下面的聊天加载器定义复制到一个本地文件中。

4. 使用指向文本文件的文件路径初始化 `WeChatChatLoader`。

5. 调用 `loader.load()`（或 `loader.lazy_load()`）进行转换。

## 1. 创建消息转储

该加载器仅支持以将应用中的消息复制到剪贴板并粘贴到文件中生成的 .txt 格式的文件。下面是一个示例。

```python
%%writefile wechat_chats.txt
女朋友 2023/09/16 2:51 PM
天气有点凉
男朋友 2023/09/16 2:51 PM
珍簟凉风著，瑶琴寄恨生。嵇君懒书札，底物慰秋情。
女朋友 2023/09/16 3:06 PM
忙什么呢
男朋友 2023/09/16 3:06 PM
今天只干成了一件像样的事
那就是想你
女朋友 2023/09/16 3:06 PM
[动画表情]
```

```output
Overwriting wechat_chats.txt
```

## 2. 定义聊天加载器

LangChain 目前不支持

```python
import logging
import re
from typing import Iterator, List
from langchain_community.chat_loaders import base as chat_loaders
from langchain_core.messages import BaseMessage, HumanMessage
logger = logging.getLogger()
class WeChatChatLoader(chat_loaders.BaseChatLoader):
    def __init__(self, path: str):
        """
        初始化微信聊天加载器。
        Args:
            path: 指向导出的微信聊天文本文件的路径。
        """
        self.path = path
        self._message_line_regex = re.compile(
            r"(?P<sender>.+?) (?P<timestamp>\d{4}/\d{2}/\d{2} \d{1,2}:\d{2} (?:AM|PM))",  # noqa
            # flags=re.DOTALL,
        )
    def _append_message_to_results(
        self,
        results: List,
        current_sender: str,
        current_timestamp: str,
        current_content: List[str],
    ):
        content = "\n".join(current_content).strip()
        # 跳过非文本消息，如贴纸、图片等。
        if not re.match(r"\[.*\]", content):
            results.append(
                HumanMessage(
                    content=content,
                    additional_kwargs={
                        "sender": current_sender,
                        "events": [{"message_time": current_timestamp}],
                    },
                )
            )
        return results
    def _load_single_chat_session_from_txt(
        self, file_path: str
    ) -> chat_loaders.ChatSession:
        """
        从文本文件中加载单个聊天会话。
        Args:
            file_path: 包含聊天消息的文本文件的路径。
        Returns:
            包含加载的聊天消息的 `ChatSession` 对象。
        """
        with open(file_path, "r", encoding="utf-8") as file:
            lines = file.readlines()
        results: List[BaseMessage] = []
        current_sender = None
        current_timestamp = None
        current_content = []
        for line in lines:
            if re.match(self._message_line_regex, line):
                if current_sender and current_content:
                    results = self._append_message_to_results(
                        results, current_sender, current_timestamp, current_content
                    )
                current_sender, current_timestamp = re.match(
                    self._message_line_regex, line
                ).groups()
                current_content = []
            else:
                current_content.append(line.strip())
        if current_sender and current_content:
            results = self._append_message_to_results(
                results, current_sender, current_timestamp, current_content
            )
        return chat_loaders.ChatSession(messages=results)
    def lazy_load(self) -> Iterator[chat_loaders.ChatSession]:
        """
        从聊天文件中延迟加载消息，并以所需格式生成它们。
        Yields:
            包含加载的聊天消息的 `ChatSession` 对象。
        """
        yield self._load_single_chat_session_from_txt(self.path)
```

## 2. 创建加载器

我们将指向刚刚写入磁盘的文件。

```python
loader = WeChatChatLoader(
    path="./wechat_chats.txt",
)
```

## 3. 加载消息

假设格式正确，加载器将把聊天转换为 LangChain 消息。

```python
from typing import List
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession
raw_messages = loader.lazy_load()
# 将同一发送者的连续消息合并为一条消息
merged_messages = merge_chat_runs(raw_messages)
# 将消息从 "男朋友" 转换为 AI 消息
messages: List[ChatSession] = list(map_ai_messages(merged_messages, sender="男朋友"))
```

```python
消息
```

```output
[{'messages': [HumanMessage(content='天气有点凉', additional_kwargs={'sender': '女朋友', 'events': [{'message_time': '2023/09/16 2:51 PM'}]}, example=False),
   AIMessage(content='珍簟凉风著，瑶琴寄恨生。嵇君懒书札，底物慰秋情。', additional_kwargs={'sender': '男朋友', 'events': [{'message_time': '2023/09/16 2:51 PM'}]}, example=False),
   HumanMessage(content='忙什么呢', additional_kwargs={'sender': '女朋友', 'events': [{'message_time': '2023/09/16 3:06 PM'}]}, example=False),
   AIMessage(content='今天只干成了一件像样的事\n那就是想你', additional_kwargs={'sender': '男朋友', 'events': [{'message_time': '2023/09/16 3:06 PM'}]}, example=False)]}]
```

### 下一步

您可以根据需要使用这些消息，例如微调模型、少样本示例选择，或直接为下一条消息做出预测  

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()
for chunk in llm.stream(messages[0]["messages"]):
    print(chunk.content, end="", flush=True)
```