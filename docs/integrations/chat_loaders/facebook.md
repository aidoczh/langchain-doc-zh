# Facebook Messenger

本文档展示了如何以可微调的格式加载来自 Facebook 的数据。总体步骤如下：

1. 将您的 Messenger 数据下载到磁盘上。

2. 创建聊天加载器并调用 `loader.load()`（或 `loader.lazy_load()`）执行转换。

3. 可选择使用 `merge_chat_runs` 将来自同一发件人的消息按顺序合并，和/或使用 `map_ai_messages` 将指定发件人的消息转换为 "AIMessage" 类。完成此操作后，调用 `convert_messages_for_finetuning` 准备数据进行微调。

完成这些步骤后，您可以微调您的模型。要完成此操作，您需要执行以下步骤：

4. 将您的消息上传到 OpenAI 并运行微调作业。

5. 在您的 LangChain 应用程序中使用生成的模型！

让我们开始吧。

## 1. 下载数据

要下载您自己的 Messenger 数据，请按照[此处的说明](https://www.zapptales.com/en/download-facebook-messenger-chat-history-how-to/)。重要提示 - 确保以 JSON 格式（而不是 HTML）下载它们。

我们在[此 Google Drive 链接](https://drive.google.com/file/d/1rh1s1o2i7B-Sk1v9o8KNgivLVGwJ-osV/view?usp=sharing)上托管了一个示例转储，我们将在本教程中使用它。

```python
# 这里使用了一些示例数据
import zipfile
import requests
def download_and_unzip(url: str, output_path: str = "file.zip") -> None:
    file_id = url.split("/")[-2]
    download_url = f"https://drive.google.com/uc?export=download&id={file_id}"
    response = requests.get(download_url)
    if response.status_code != 200:
        print("文件下载失败。")
        return
    with open(output_path, "wb") as file:
        file.write(response.content)
        print(f"文件 {output_path} 已下载。")
    with zipfile.ZipFile(output_path, "r") as zip_ref:
        zip_ref.extractall()
        print(f"文件 {output_path} 已解压。")
# 要下载的文件的 URL
url = "https://drive.google.com/file/d/1rh1s1o2i7B-Sk1v9o8KNgivLVGwJ-osV/view?usp=sharing"
# 下载并解压
download_and_unzip(url)
```

```output
文件 file.zip 已下载。
文件 file.zip 已解压。
```

## 2. 创建聊天加载器

我们有两种不同的 `FacebookMessengerChatLoader` 类，一种用于整个聊天目录，另一种用于加载单个文件。

```python
directory_path = "./hogwarts"
```

```python
from langchain_community.chat_loaders.facebook_messenger import (
    FolderFacebookMessengerChatLoader,
    SingleFileFacebookMessengerChatLoader,
)
```

```python
loader = SingleFileFacebookMessengerChatLoader(
    path="./hogwarts/inbox/HermioneGranger/messages_Hermione_Granger.json",
)
```

```python
chat_session = loader.load()[0]
chat_session["messages"][:3]
```

```output
[HumanMessage(content="Hi Hermione! How's your summer going so far?", additional_kwargs={'sender': 'Harry Potter'}),
 HumanMessage(content="Harry! Lovely to hear from you. My summer is going well, though I do miss everyone. I'm spending most of my time going through my books and researching fascinating new topics. How about you?", additional_kwargs={'sender': 'Hermione Granger'}),
 HumanMessage(content="I miss you all too. The Dursleys are being their usual unpleasant selves but I'm getting by. At least I can practice some spells in my room without them knowing. Let me know if you find anything good in your researching!", additional_kwargs={'sender': 'Harry Potter'})]
```

```python
loader = FolderFacebookMessengerChatLoader(
    path="./hogwarts",
)
```

```python
chat_sessions = loader.load()
len(chat_sessions)
```

```output
9
```

## 3. 准备微调

调用 `load()` 返回我们可以提取的所有聊天消息作为人类消息。与真实对话相比，与聊天机器人交谈时，对话通常遵循更严格的交替对话模式。

您可以选择合并消息 "runs"（来自同一发件人的连续消息），并选择一个发件人来代表 "AI"。微调的 LLM 将学习生成这些 AI 消息。

```python
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
```

```python
merged_sessions = merge_chat_runs(chat_sessions)
alternating_sessions = list(map_ai_messages(merged_sessions, "Harry Potter"))
```

```python
# 现在所有哈利·波特的消息将采用 AI 消息类，该类映射到 OpenAI 的训练格式中的 'assistant' 角色
alternating_sessions[0]["messages"][:3]
```

```output
[AIMessage(content="Professor Snape, I was hoping I could speak with you for a moment about something that's been concerning me lately.", additional_kwargs={'sender': 'Harry Potter'}),
 HumanMessage(content="What is it, Potter? I'm quite busy at the moment.", additional_kwargs={'sender': 'Severus Snape'}),
 AIMessage(content="I apologize for the interruption, sir. I'll be brief. I've noticed some strange activity around the school grounds at night. I saw a cloaked figure lurking near the Forbidden Forest last night. I'm worried someone may be plotting something sinister.", additional_kwargs={'sender': 'Harry Potter'})]
```

#### 现在我们可以将其转换为 OpenAI 格式的字典

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning
```

```python
training_data = convert_messages_for_finetuning(alternating_sessions)
print(f"准备了 {len(training_data)} 个对话进行训练")
```

```output
准备了 9 个对话进行训练
```

```python
training_data[0][:3]
```

```output
[{'role': 'assistant',
  'content': "Snape 教授，我希望能和您谈谈最近让我担心的事情。"},
 {'role': 'user',
  'content': "什么事，波特？我现在很忙。"},
 {'role': 'assistant',
  'content': "对不起打扰您，先生。我会说得很快。我最近注意到学校周围有一些奇怪的活动。昨晚我看到一个披着斗篷的人在禁忌森林附近潜伏。我担心有人可能正在策划一些邪恶的事情。"}]
```

目前，OpenAI 要求至少有 10 个训练样本进行微调，尽管他们建议大多数任务使用 50-100 个样本。由于我们只有 9 个聊天会话，我们可以将它们细分（可选地有一些重叠），以便每个训练样本由整个对话的一部分组成。

Facebook 的聊天会话（每人一个）通常跨越多天和多个对话，

因此长期依赖关系可能并不重要。

```python
# 我们的聊天是交替进行的，我们将每个数据点设置为 8 条消息，
# 重叠 2 条消息
chunk_size = 8
overlap = 2
training_examples = [
    conversation_messages[i : i + chunk_size]
    for conversation_messages in training_data
    for i in range(0, len(conversation_messages) - chunk_size + 1, chunk_size - overlap)
]
len(training_examples)
```

```output
100
```

## 4. 对模型进行微调

现在是时候对模型进行微调了。确保已安装 `openai`

并正确设置了 `OPENAI_API_KEY`

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
for m in training_examples:
    my_file.write((json.dumps({"messages": m}) + "\n").encode("utf-8"))
my_file.seek(0)
training_file = openai.files.create(file=my_file, purpose="fine-tune")
# 出于合规性原因，OpenAI 会审核每个训练文件。
# 这可能需要几分钟的时间
status = openai.files.retrieve(training_file.id).status
start_time = time.time()
while status != "processed":
    print(f"状态=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    status = openai.files.retrieve(training_file.id).status
print(f"文件 {training_file.id} 在 {time.time() - start_time:.2f} 秒后准备好。")
```

```output
文件 file-ULumAXLEFw3vB6bb9uy6DNVC 在 0.00 秒后准备好。
```

文件准备好后，是时候启动训练作业了。

```python
job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)
```

在模型准备好之前，喝杯茶吧。这可能需要一些时间！

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
状态=[running]... 874.29s. 56.93s
```

```python
print(job.fine_tuned_model)
```

```output
ft:gpt-3.5-turbo-0613:personal::8QnAzWMr
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
        ("human", "{input}"),
    ]
)
chain = prompt | model | StrOutputParser()
```

```python
for tok in chain.stream({"input": "你正在上哪些课程？"}):
    print(tok, end="", flush=True)
```

```output
我正在上魔法学、黑魔法防御术、草药学、魔药学、变形术和古代符文学。你呢？
```