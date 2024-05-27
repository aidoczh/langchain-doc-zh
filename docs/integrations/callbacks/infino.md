# Infino

[Infino](https://github.com/infinohq/infino) 是一个可扩展的遥测存储系统，专为日志、指标和跟踪而设计。Infino可以作为独立的可观测性解决方案，也可以作为您的可观测性堆栈中的存储层。

这个例子展示了如何在通过 `LangChain` 和 [Infino](https://github.com/infinohq/infino) 调用 OpenAI 和 ChatOpenAI 模型时跟踪以下内容：

- 提示输入

- `ChatGPT` 或任何其他 `LangChain` 模型的响应

- 延迟

- 错误

- 消耗的标记数量

## 初始化

```python
# 安装必要的依赖项。
%pip install --upgrade --quiet  infinopy
%pip install --upgrade --quiet  matplotlib
%pip install --upgrade --quiet  tiktoken
```

```python
from langchain_community.callbacks.infino_callback import InfinoCallbackHandler
```

```python
import datetime as dt
import json
import time
import matplotlib.dates as md
import matplotlib.pyplot as plt
from infinopy import InfinoClient
from langchain_openai import OpenAI
```

## 启动 Infino 服务器，初始化 Infino 客户端

```python
# 使用 Infino docker 镜像启动服务器。
!docker run --rm --detach --name infino-example -p 3000:3000 infinohq/infino:latest
# 创建 Infino 客户端。
client = InfinoClient()
```

```output
a1159e99c6bdb3101139157acee6aba7ae9319375e77ab6fbc79beff75abeca3
```

## 读取问题数据集

```python
# 这些是来自斯坦福问答数据集的一部分问题 -
# https://rajpurkar.github.io/SQuAD-explorer/
data = """In what country is Normandy located?
When were the Normans in Normandy?
From which countries did the Norse originate?
Who was the Norse leader?
What century did the Normans first gain their separate identity?
Who gave their name to Normandy in the 1000's and 1100's
What is France a region of?
Who did King Charles III swear fealty to?
When did the Frankish identity emerge?
Who was the duke in the battle of Hastings?
Who ruled the duchy of Normandy
What religion were the Normans
What type of major impact did the Norman dynasty have on modern Europe?
Who was famed for their Christian spirit?
Who assimilted the Roman language?
Who ruled the country of Normandy?
What principality did William the conqueror found?
What is the original meaning of the word Norman?
When was the Latin version of the word Norman first recorded?
What name comes from the English words Normans/Normanz?"""
questions = data.split("\n")
```

## 示例 1: LangChain OpenAI 问答; 将指标和日志发布到 Infino

```python
# 在这里设置您的密钥。
# os.environ["OPENAI_API_KEY"] = "YOUR_API_KEY"
# 创建回调处理程序。这将记录延迟、错误、标记使用情况、提示以及提示响应到 Infino。
handler = InfinoCallbackHandler(
    model_id="test_openai", model_version="0.1", verbose=False
)
# 创建 LLM。
llm = OpenAI(temperature=0.1)
# 要向 OpenAI 模型提问的问题数量。在这里我们限制了一个较短的数量，以节省费用，同时运行此演示。
num_questions = 10
questions = questions[0:num_questions]
for question in questions:
    print(question)
    # 我们使用 Infino 回调将问题发送到 OpenAI API。
    llm_result = llm.generate([question], callbacks=[handler])
    print(llm_result)
```

```output
In what country is Normandy located?
generations=[[Generation(text='\n\nNormandy is located in France.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 16, 'prompt_tokens': 7, 'completion_tokens': 9}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('67a516e3-d48a-4e83-92ba-a139079bd3b1'))]
When were the Normans in Normandy?
generations=[[Generation(text='\n\nThe Normans first settled in Normandy in the late 9th century.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 24, 'prompt_tokens': 8, 'completion_tokens': 16}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('6417a773-c863-4942-9607-c8a0c5d486e7'))]
From which countries did the Norse originate?
generations=[[Generation(text='\n\nThe Norse originated from Scandinavia, which includes the modern-day countries of Norway, Sweden, and Denmark.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 32, 'prompt_tokens': 8, 'completion_tokens': 24}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('70547d72-7925-454e-97fb-5539f8788c3f'))]
Who was the Norse leader?
generations=[[Generation(text='\n\nThe most famous Norse leader was the legendary Viking king Ragnar Lodbrok. He was a legendary Viking hero and ruler who is said to have lived in the 9th century. He is known for his legendary exploits, including leading a Viking raid on Paris in 845.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 62, 'prompt_tokens': 6, 'completion_tokens': 56}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('04500e37-44ab-4e56-9017-76fe8c19e2ca'))]
What century did the Normans first gain their separate identity?
generations=[[Generation(text='\n\nThe Normans first gained their separate identity in the 11th century.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 28, 'prompt_tokens': 12, 'completion_tokens': 16}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('adf319b7-1022-40df-9afe-1d65f869d83d'))]
Who gave their name to Normandy in the 1000's and 1100's
generations=[[Generation(text='\n\nThe Normans, a people from northern France, gave their name to Normandy in the 1000s and 1100s. The Normans were descendants of Vikings who had settled in the region in the late 800s.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 57, 'prompt_tokens': 13, 'completion_tokens': 44}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('1a0503bc-d033-4b69-a5fa-5e1796566133'))]
What is France a region of?
generations=[[Generation(text='\n\nFrance is a region of Europe.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 16, 'prompt_tokens': 7, 'completion_tokens': 9}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('7485d954-1c14-4dff-988a-25a0aa0871cc'))]
Who did King Charles III swear fealty to?
generations=[[Generation(text='\n\nKing Charles III swore fealty to King Philip II of Spain.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 25, 'prompt_tokens': 10, 'completion_tokens': 15}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('292c7143-4a08-43cd-a1e1-42cb1f594f33'))]
When did the Frankish identity emerge?
generations=[[Generation(text='\n\nThe Frankish identity began to emerge in the late 5th century, when the Franks began to expand their power and influence in the region. The Franks were a Germanic tribe that had settled in the area of modern-day France and Germany. They eventually established the Merovingian dynasty, which ruled much of Western Europe from the mid-6th century until 751.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 85, 'prompt_tokens': 8, 'completion_tokens': 77}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('3d9475c2-931e-4217-8bc3-b3e970e7597c'))]
Who was the duke in the battle of Hastings?
generations=[[Generation(text='\n\nThe Duke of Normandy, William the Conqueror, was the leader of the Norman forces at the Battle of Hastings in 1066.', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 39, 'prompt_tokens': 11, 'completion_tokens': 28}, 'model_name': 'text-davinci-003'} run=[RunInfo(run_id=UUID('b8f84619-ea5f-4c18-b411-b62194f36fe0'))]
```

## 创建度量图表

现在我们使用 matplotlib 创建延迟、错误和消耗的令牌的图表。

```python
# 使用 matplotlib 创建图表的辅助函数
def plot(data, title):
    data = json.loads(data)
    # 从数据中提取 x 和 y 值
    timestamps = [item["time"] for item in data]
    dates = [dt.datetime.fromtimestamp(ts) for ts in timestamps]
    y = [item["value"] for item in data]
    plt.rcParams["figure.figsize"] = [6, 4]
    plt.subplots_adjust(bottom=0.2)
    plt.xticks(rotation=25)
    ax = plt.gca()
    xfmt = md.DateFormatter("%Y-%m-%d %H:%M:%S")
    ax.xaxis.set_major_formatter(xfmt)
    # 创建图表
    plt.plot(dates, y)
    # 设置标签和标题
    plt.xlabel("时间")
    plt.ylabel("数值")
    plt.title(title)
    plt.show()
```

```python
response = client.search_ts("__name__", "latency", 0, int(time.time()))
plot(response.text, "延迟")
response = client.search_ts("__name__", "error", 0, int(time.time()))
plot(response.text, "错误")
response = client.search_ts("__name__", "prompt_tokens", 0, int(time.time()))
plot(response.text, "提示令牌")
response = client.search_ts("__name__", "completion_tokens", 0, int(time.time()))
plot(response.text, "完成令牌")
response = client.search_ts("__name__", "total_tokens", 0, int(time.time()))
plot(response.text, "总令牌")
```

## 对提示或提示输出进行全文查询

```python
# 搜索特定提示文本
query = "诺曼底"
response = client.search_log(query, 0, int(time.time()))
print("关于", query, "的结果：", response.text)
print("===")
query = "查理三世"
response = client.search_log("查理三世", 0, int(time.time()))
print("关于", query, "的结果：", response.text)
```

```output
关于 诺曼底 的结果：[{"time":1696947743,"fields":{"prompt_response":"\n\n诺曼底是法国北部的一个地区，得名于公元1000年和1100年间的诺曼人。诺曼人是维京人的后裔，他们在公元800年代定居在该地区。"},"text":"\n\n诺曼底是法国北部的一个地区，得名于公元1000年和1100年间的诺曼人。诺曼人是维京人的后裔，他们在公元800年代定居在该地区。"},{"time":1696947740,"fields":{"prompt":"在1000年代和1100年代，谁使诺曼底得名"},"text":"在1000年代和1100年代，谁使诺曼底得名"},{"time":1696947733,"fields":{"prompt_response":"\n\n诺曼人最初定居在诺曼底是在9世纪末。"},"text":"\n\n诺曼人最初定居在诺曼底是在9世纪末。"},{"time":1696947732,"fields":{"prompt_response":"\n\n诺曼底位于法国。"},"text":"\n\n诺曼底位于法国。"},{"time":1696947731,"fields":{"prompt":"诺曼底位于哪个国家？"},"text":"诺曼底位于哪个国家？"}]
===
关于 查理三世 的结果：[{"time":1696947745,"fields":{"prompt_response":"\n\n查理三世向西班牙国王菲利普二世宣誓效忠。"},"text":"\n\n查理三世向西班牙国王菲利普二世宣誓效忠。"},{"time":1696947744,"fields":{"prompt":"查理三世向谁宣誓效忠？"},"text":"查理三世向谁宣誓效忠？"}]
```

# 例子 2：使用 ChatOpenAI 对一段文本进行总结

```python
# 在此设置你的密钥。
# os.environ["OPENAI_API_KEY"] = "YOUR_API_KEY"
from langchain.chains.summarize import load_summarize_chain
from langchain_community.document_loaders import WebBaseLoader
from langchain_openai import ChatOpenAI
# 创建回调处理程序。这将记录延迟、错误、令牌使用情况、提示以及对提示的响应到 Infino。
handler = InfinoCallbackHandler(
    model_id="test_chatopenai", model_version="0.1", verbose=False
)
urls = [
    "https://lilianweng.github.io/posts/2023-06-23-agent/",
    "https://medium.com/lyft-engineering/lyftlearn-ml-model-training-infrastructure-built-on-kubernetes-aef8218842bb",
    "https://blog.langchain.dev/week-of-10-2-langchain-release-notes/",
]
for url in urls:
    loader = WebBaseLoader(url)
    docs = loader.load()
    llm = ChatOpenAI(temperature=0, model_name="gpt-3.5-turbo-16k", callbacks=[handler])
    chain = load_summarize_chain(llm, chain_type="stuff", verbose=False)
    chain.run(docs)
```

## 创建度量图表

```python
response = client.search_ts("__name__", "latency", 0, int(time.time()))
plot(response.text, "延迟")
response = client.search_ts("__name__", "error", 0, int(time.time()))
plot(response.text, "错误")
response = client.search_ts("__name__", "prompt_tokens", 0, int(time.time()))
plot(response.text, "提示令牌")
response = client.search_ts("__name__", "completion_tokens", 0, int(time.time()))
plot(response.text, "完成令牌")
```

```python
查询 = "机器学习"
回应 = 客户端.搜索日志(查询, 0, int(time.time()))
# 输出可能很冗长 - 如果需要打印，请取消下面的注释。
# print("Results for", query, ":", response.text)
print("===")
```

```output
===
```

```python
## 停止 Infino 服务器
```

```python
!docker rm -f infino-example
```

```output
infino-example
```