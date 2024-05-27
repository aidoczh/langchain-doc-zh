# iMessage

这本笔记本展示了如何使用 iMessage 聊天加载器。这个类帮助将 iMessage 对话转换为 LangChain 聊天消息。

在 MacOS 上，iMessage 将对话存储在 `~/Library/Messages/chat.db` 的 sqlite 数据库中（至少对于 macOS Ventura 13.4 是这样）。

`IMessageChatLoader` 从这个数据库文件加载。

1. 使用指向要处理的 `chat.db` 数据库的文件路径创建 `IMessageChatLoader`。

2. 调用 `loader.load()`（或 `loader.lazy_load()`）执行转换。可选择使用 `merge_chat_runs` 将来自同一发送者的消息合并在一起，和/或使用 `map_ai_messages` 将来自指定发送者的消息转换为 "AIMessage" 类。

## 1. 访问聊天数据库

你的终端可能被拒绝访问 `~/Library/Messages`。为了使用这个类，你可以将数据库复制到一个可访问的目录（例如文档）并从那里加载。另外（并且不建议），你可以在系统设置 > 安全性与隐私 > 完全磁盘访问 中为你的终端模拟器授予完全磁盘访问权限。

我们已经创建了一个示例数据库，你可以在[此链接的云端文件](https://drive.google.com/file/d/1NebNKqTA2NXApCmeH6mu0unJD2tANZzo/view?usp=sharing)中使用。

```python
# 这里使用了一些示例数据
import requests
def download_drive_file(url: str, output_path: str = "chat.db") -> None:
    file_id = url.split("/")[-2]
    download_url = f"https://drive.google.com/uc?export=download&id={file_id}"
    response = requests.get(download_url)
    if response.status_code != 200:
        print("文件下载失败。")
        return
    with open(output_path, "wb") as file:
        file.write(response.content)
        print(f"文件 {output_path} 已下载。")
url = "https://drive.google.com/file/d/1NebNKqTA2NXApCmeH6mu0unJD2tANZzo/view?usp=sharing"
# 下载文件到 chat.db
download_drive_file(url)
```

```output
文件 chat.db 已下载。
```

## 2. 创建聊天加载器

提供加载器与 zip 目录的文件路径。你可以选择指定映射到 AI 消息的用户 id，以及配置是否合并消息运行。

```python
from langchain_community.chat_loaders.imessage import IMessageChatLoader
```

```python
loader = IMessageChatLoader(
    path="./chat.db",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回一个当前仅包含每个加载对话的消息列表的 "ChatSessions" 列表。所有消息最初都映射到 "HumanMessage" 对象。

你可以选择合并消息 "runs"（来自同一发送者的连续消息）并选择一个发送者来代表 "AI"。经过精调的 LLM 将学习生成这些 AI 消息。

```python
from typing import List
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession
raw_messages = loader.lazy_load()
# 将来自同一发送者的连续消息合并为单个消息
merged_messages = merge_chat_runs(raw_messages)
# 将消息从 "Tortoise" 转换为 AI 消息。你能猜到这些对话是谁之间的吗？
chat_sessions: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="Tortoise")
)
```

```python
# 现在所有的 Tortoise 的消息将采用 AI 消息类
# 这对应于 OpenAI 训练格式中的 'assistant' 角色
chat_sessions[0]["messages"][:3]
```

```output
[AIMessage(content="Slow and steady, that's my motto.", additional_kwargs={'message_time': 1693182723, 'sender': 'Tortoise'}, example=False),
 HumanMessage(content='Speed is key!', additional_kwargs={'message_time': 1693182753, 'sender': 'Hare'}, example=False),
 AIMessage(content='A balanced approach is more reliable.', additional_kwargs={'message_time': 1693182783, 'sender': 'Tortoise'}, example=False)]
```

## 3. 准备进行精调

现在是时候将我们的聊天消息转换为 OpenAI 字典了。我们可以使用 `convert_messages_for_finetuning` 实用程序来完成。

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning
```

```python
training_data = convert_messages_for_finetuning(chat_sessions)
print(f"为训练准备了 {len(training_data)} 个对话")
```

```output
为训练准备了 10 个对话
```

## 4. 对模型进行精调

是时候对模型进行精调了。确保你已安装了 `openai` 并且已经适当设置了你的 `OPENAI_API_KEY`。

```python
%pip install --upgrade --quiet  langchain-openai
```

```python
import json
import time
from io import BytesIO
import openai
# 我们将在内存中写入 jsonl 文件
my_file = BytesIO()
for m in training_data:
    my_file.write((json.dumps({"messages": m}) + "\n").encode("utf-8"))
my_file.seek(0)
training_file = openai.files.create(file=my_file, purpose="fine-tune")
# OpenAI 会出于合规性原因审计每个训练文件。
# 这可能需要几分钟
status = openai.files.retrieve(training_file.id).status
start_time = time.time()
while status != "processed":
    print(f"状态=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    status = openai.files.retrieve(training_file.id).status
print(f"文件 {training_file.id} 在 {time.time() - start_time:.2f} 秒后准备好了。")
```

```output
文件 file-zHIgf4r8LltZG3RFpkGd4Sjf 在 10.19 秒后准备就绪。
```

文件准备就绪后，是时候开始训练工作了。

```python
job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)
```

在模型准备过程中，可以泡杯茶放松一下。这可能需要一些时间！

```python
status = openai.fine_tuning.jobs.retrieve(job.id).status
start_time = time.time()
while status != "succeeded":
    print(f"状态=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    job = openai.fine_tuning.jobs.retrieve(job.id)
    status = job.status
```

```output
状态=[running]... 524.95s
```

```python
print(job.fine_tuned_model)
```

```output
ft:gpt-3.5-turbo-0613:personal::7sKoRdlz
```

## 5. 在 LangChain 中使用

您可以直接使用生成的模型 ID 来调用 `ChatOpenAI` 模型类。

```python
from langchain_openai import ChatOpenAI
model = ChatOpenAI(
    model=job.fine_tuned_model,
    temperature=1,
)
```

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are speaking to hare."),
        ("human", "{input}"),
    ]
)
chain = prompt | model | StrOutputParser()
```

```python
for tok in chain.stream({"input": "What's the golden thread?"}):
    print(tok, end="", flush=True)
```

```output
一种相互关联的象征。
```