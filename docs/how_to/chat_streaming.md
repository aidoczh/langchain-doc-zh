---
sidebar_position: 1.5
---
# 如何流式传输聊天模型的响应
所有[聊天模型](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html)都实现了[Runnable接口](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable)，该接口提供了标准可运行方法（即`ainvoke`、`batch`、`abatch`、`stream`、`astream`、`astream_events`）的**默认**实现。
**默认**的流式传输实现提供了一个`Iterator`（或用于异步流式传输的`AsyncIterator`），它产生一个单一值：来自底层聊天模型提供程序的最终输出。
:::tip
**默认**实现**不**支持逐标记的流式传输，但它确保模型可以替换为任何其他模型，因为它支持相同的标准接口。
:::
逐标记流式传输的能力取决于提供程序是否实现了适当的流式传输支持。
查看哪些[集成支持逐标记流式传输在这里](/docs/integrations/chat/)。
## 同步流式传输
下面我们使用`|`来帮助可视化标记之间的分隔符。
```python
from langchain_anthropic.chat_models import ChatAnthropic
chat = ChatAnthropic(model="claude-3-haiku-20240307")
for chunk in chat.stream("Write me a 1 verse song about goldfish on the moon"):
    print(chunk.content, end="|", flush=True)
```
```output
Here| is| a| |1| |verse| song| about| gol|dfish| on| the| moon|:|
Floating| up| in| the| star|ry| night|,|
Fins| a|-|gl|im|mer| in| the| pale| moon|light|.|
Gol|dfish| swimming|,| peaceful| an|d free|,|
Se|ren|ely| |drif|ting| across| the| lunar| sea|.|
```
## 异步流式传输
```python
from langchain_anthropic.chat_models import ChatAnthropic
chat = ChatAnthropic(model="claude-3-haiku-20240307")
async for chunk in chat.astream("Write me a 1 verse song about goldfish on the moon"):
    print(chunk.content, end="|", flush=True)
```
```output
Here| is| a| |1| |verse| song| about| gol|dfish| on| the| moon|:|
Floating| up| above| the| Earth|,|
Gol|dfish| swim| in| alien| m|irth|.|
In| their| bowl| of| lunar| dust|,|
Gl|it|tering| scales| reflect| the| trust|
Of| swimming| free| in| this| new| worl|d,|
Where| their| aqu|atic| dream|'s| unf|ur|le|d.|
```
## Astream 事件
聊天模型还支持标准的[astream events](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events)方法。
如果您正在从包含多个步骤的较大LLM应用程序中流式传输输出（例如，由提示、llm和解析器组成的LLM链），则此方法很有用。
```python
from langchain_anthropic.chat_models import ChatAnthropic
chat = ChatAnthropic(model="claude-3-haiku-20240307")
idx = 0
async for event in chat.astream_events(
    "Write me a 1 verse song about goldfish on the moon", version="v1"
):
    idx += 1
    if idx >= 5:  # 截断输出
        print("...已截断")
        break
    print(event)
```
```output
{'event': 'on_chat_model_start', 'run_id': '08da631a-12a0-4f07-baee-fc9a175ad4ba', 'name': 'ChatAnthropic', 'tags': [], 'metadata': {}, 'data': {'input': 'Write me a 1 verse song about goldfish on the moon'}}
{'event': 'on_chat_model_stream', 'run_id': '08da631a-12a0-4f07-baee-fc9a175ad4ba', 'tags': [], 'metadata': {}, 'name': 'ChatAnthropic', 'data': {'chunk': AIMessageChunk(content='Here', id='run-08da631a-12a0-4f07-baee-fc9a175ad4ba')}}
{'event': 'on_chat_model_stream', 'run_id': '08da631a-12a0-4f07-baee-fc9a175ad4ba', 'tags': [], 'metadata': {}, 'name': 'ChatAnthropic', 'data': {'chunk': AIMessageChunk(content="'s", id='run-08da631a-12a0-4f07-baee-fc9a175ad4ba')}}
{'event': 'on_chat_model_stream', 'run_id': '08da631a-12a0-4f07-baee-fc9a175ad4ba', 'tags': [], 'metadata': {}, 'name': 'ChatAnthropic', 'data': {'chunk': AIMessageChunk(content=' a', id='run-08da631a-12a0-4f07-baee-fc9a175ad4ba')}}
...已截断
```