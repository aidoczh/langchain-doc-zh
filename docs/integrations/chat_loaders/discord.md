# Discord
本笔记本展示了如何创建自己的聊天加载器，该加载器可以将从 Discord 私信中复制粘贴的消息转换为 LangChain 消息列表。
该过程分为四步：
1. 通过从 Discord 应用程序中复制聊天内容并将其粘贴到本地计算机上的文件中来创建聊天 .txt 文件。
2. 从下面复制聊天加载器定义到本地文件。
3. 使用指向文本文件的文件路径初始化 `DiscordChatLoader`。
4. 调用 `loader.load()`（或 `loader.lazy_load()`）来执行转换。
## 1. 创建消息转储
目前（2023/08/23），该加载器仅支持通过将应用程序中的消息复制到剪贴板并粘贴到文件中生成的 .txt 格式文件。以下是一个示例。
```python
%%writefile discord_chats.txt
talkingtower — 08/15/2023 11:10 AM
喜欢音乐！你喜欢爵士乐吗？
reporterbob — 08/15/2023 9:27 PM
是的！爵士乐太棒了。听过这首吗？
网站
收听经典爵士乐曲...
talkingtower — 昨天上午5:03
确实！很棒的选择。🎷
reporterbob — 昨天上午5:23
谢谢！怎么样来一场虚拟观光？
网站
著名地标的虚拟游览...
talkingtower — 今天下午2:38
听起来很有趣！让我们来探索一下。
reporterbob — 今天下午2:56
享受旅行！待会见。
talkingtower — 今天下午3:00
谢谢！再见！👋
reporterbob — 今天下午3:02
再会！祝你探险愉快。
```
```output
Writing discord_chats.txt
```
## 2. 定义聊天加载器
```python
import logging
import re
from typing import Iterator, List
from langchain_community.chat_loaders import base as chat_loaders
from langchain_core.messages import BaseMessage, HumanMessage
logger = logging.getLogger()
class DiscordChatLoader(chat_loaders.BaseChatLoader):
    def __init__(self, path: str):
        """
        初始化 Discord 聊天加载器。
        Args:
            path: 指向导出的 Discord 聊天文本文件的路径。
        """
        self.path = path
        self._message_line_regex = re.compile(
            r"(.+?) — (\w{3,9} \d{1,2}(?:st|nd|rd|th)?(?:, \d{4})? \d{1,2}:\d{2} (?:AM|PM)|Today at \d{1,2}:\d{2} (?:AM|PM)|Yesterday at \d{1,2}:\d{2} (?:AM|PM))",  # noqa
            flags=re.DOTALL,
        )
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
            if re.match(
                r".+? — (\d{2}/\d{2}/\d{4} \d{1,2}:\d{2} (?:AM|PM)|Today at \d{1,2}:\d{2} (?:AM|PM)|Yesterday at \d{1,2}:\d{2} (?:AM|PM))",  # noqa
                line,
            ):
                if current_sender and current_content:
                    results.append(
                        HumanMessage(
                            content="".join(current_content).strip(),
                            additional_kwargs={
                                "sender": current_sender,
                                "events": [{"message_time": current_timestamp}],
                            },
                        )
                    )
                current_sender, current_timestamp = line.split(" — ")[:2]
                current_content = [
                    line[len(current_sender) + len(current_timestamp) + 4 :].strip()
                ]
            elif re.match(r"\[\d{1,2}:\d{2} (?:AM|PM)\]", line.strip()):
                results.append(
                    HumanMessage(
                        content="".join(current_content).strip(),
                        additional_kwargs={
                            "sender": current_sender,
                            "events": [{"message_time": current_timestamp}],
                        },
                    )
                )
                current_timestamp = line.strip()[1:-1]
                current_content = []
            else:
                current_content.append("\n" + line.strip())
        if current_sender and current_content:
            results.append(
                HumanMessage(
                    content="".join(current_content).strip(),
                    additional_kwargs={
                        "sender": current_sender,
                        "events": [{"message_time": current_timestamp}],
                    },
                )
            )
        return chat_loaders.ChatSession(messages=results)
    def lazy_load(self) -> Iterator[chat_loaders.ChatSession]:
        """
        从聊天文件中延迟加载消息并以所需格式进行迭代。
        Yields:
            包含加载的聊天消息的 `ChatSession` 对象。
        """
        yield self._load_single_chat_session_from_txt(self.path)
```
## 2. 创建加载器
我们将指向我们刚刚写入磁盘的文件。
```python
loader = DiscordChatLoader(
    path="./discord_chats.txt",
)
```
## 3. 加载消息
假设格式正确，加载器将把聊天转换为 langchain 消息。
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
# 将消息从 "talkingtower" 转换为 AI 消息
messages: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="talkingtower")
)
```
```python
messages
```
```output
[{'messages': [AIMessage(content='喜欢音乐！你喜欢爵士乐吗？', additional_kwargs={'sender': 'talkingtower', 'events': [{'message_time': '08/15/2023 11:10 AM\n'}]}),
   HumanMessage(content='是的！爵士乐太棒了。听过这首吗？\n网址\n听经典爵士乐曲...', additional_kwargs={'sender': 'reporterbob', 'events': [{'message_time': '08/15/2023 9:27 PM\n'}]}),
   AIMessage(content='确实！好选择。🎷', additional_kwargs={'sender': 'talkingtower', 'events': [{'message_time': '昨天上午5:03\n'}]}),
   HumanMessage(content='谢谢！来虚拟观光一下怎么样？\n网址\n虚拟参观著名地标...', additional_kwargs={'sender': 'reporterbob', 'events': [{'message_time': '昨天上午5:23\n'}]}),
   AIMessage(content='听起来很有趣！我们一起探索吧。', additional_kwargs={'sender': 'talkingtower', 'events': [{'message_time': '今天下午2:38\n'}]}),
   HumanMessage(content='享受观光！再见。', additional_kwargs={'sender': 'reporterbob', 'events': [{'message_time': '今天下午2:56\n'}]}),
   AIMessage(content='谢谢！再见！👋', additional_kwargs={'sender': 'talkingtower', 'events': [{'message_time': '今天下午3:00\n'}]}),
   HumanMessage(content='再见！祝你探索愉快。', additional_kwargs={'sender': 'reporterbob', 'events': [{'message_time': '今天下午3:02\n'}]})]}]
```
### 下一步
然后，您可以根据需要使用这些消息，例如微调模型、少样本示例选择或直接对下一条消息进行预测。
```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI()
for chunk in llm.stream(messages[0]["messages"]):
    print(chunk.content, end="", flush=True)
```
```output
谢谢！祝您有美好的一天！
```