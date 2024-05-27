# 如何从 LLM 流式传输响应
所有的 `LLM` 都实现了 [Runnable 接口](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable)，其中包含了标准可运行方法的**默认**实现（即 `ainvoke`、`batch`、`abatch`、`stream`、`astream`、`astream_events`）。
**默认**的流式传输实现提供了一个`Iterator`（或者对于异步流式传输来说是 `AsyncIterator`），它会产生一个单一的值：来自底层聊天模型提供者的最终输出。
逐个令牌地流式传输输出的能力取决于提供者是否实现了适当的流式传输支持。
查看哪些[集成支持逐个令牌的流式传输](/docs/integrations/llms/)。
:::note
**默认**实现**不支持**逐个令牌的流式传输，但它确保模型可以替换为任何其他模型，因为它支持相同的标准接口。
:::
## 同步流式传输
下面的示例中，我们使用 `|` 来可视化令牌之间的分隔符。
```python
from langchain_openai import OpenAI
llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)
for chunk in llm.stream("Write me a 1 verse song about sparkling water."):
    print(chunk, end="|", flush=True)
```
```output
|Spark|ling| water|,| oh| so clear||Bubbles dancing|,| without| fear||Refreshing| taste|,| a| pure| delight||Spark|ling| water|,| my| thirst|'s| delight||
```
## 异步流式传输
让我们看看如何在异步环境中使用 `astream` 进行流式传输。
```python
from langchain_openai import OpenAI
llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)
async for chunk in llm.astream("Write me a 1 verse song about sparkling water."):
    print(chunk, end="|", flush=True)
```
```output
|Spark|ling| water|,| oh| so clear||Bubbles dancing|,| without| fear||Refreshing| taste|,| a| pure| delight||Spark|ling| water|,| my| thirst|'s| delight||
```
## 异步事件流式传输
LLM 也支持标准的 [astream events](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events) 方法。
:::tip
在实现包含多个步骤的流式传输的较大 LLM 应用程序中（例如涉及 `agent` 的应用程序），`astream_events` 最有用。
:::
```python
from langchain_openai import OpenAI
llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)
idx = 0
async for event in llm.astream_events(
    "Write me a 1 verse song about goldfish on the moon", version="v1"
):
    idx += 1
    if idx >= 5:  # 截断输出
        print("...Truncated")
        break
    print(event)
```