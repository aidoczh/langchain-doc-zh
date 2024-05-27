# 如何流式运行
:::info 前提条件
本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [LangChain 表达式语言](/docs/concepts/#langchain-expression-language)
- [输出解析器](/docs/concepts/#output-parsers)
:::
流式运行对于使基于 LLM 的应用程序对最终用户具有响应性至关重要。
重要的 LangChain 原语，如[聊天模型](/docs/concepts/#chat-models)、[输出解析器](/docs/concepts/#output-parsers)、[提示模板](/docs/concepts/#prompt-templates)、[检索器](/docs/concepts/#retrievers)和[代理](/docs/concepts/#agents)都实现了 LangChain [Runnable 接口](/docs/concepts#interface)。
该接口提供了两种通用的流式内容方法：
1. 同步 `stream` 和异步 `astream`：流式传输链中的**最终输出**的**默认实现**。
2. 异步 `astream_events` 和异步 `astream_log`：这些方法提供了一种从链中流式传输**中间步骤**和**最终输出**的方式。
让我们看看这两种方法，并尝试理解如何使用它们。
## 使用 Stream
所有 `Runnable` 对象都实现了一个名为 `stream` 的同步方法和一个名为 `astream` 的异步变体。
这些方法旨在以块的形式流式传输最终输出，尽快返回每个块。
只有在程序中的所有步骤都知道如何处理**输入流**时，才能进行流式传输；即，逐个处理输入块，并产生相应的输出块。
这种处理的复杂性可以有所不同，从简单的任务，如发出 LLM 生成的令牌，到更具挑战性的任务，如在整个 JSON 完成之前流式传输 JSON 结果的部分。
开始探索流式传输的最佳方法是从 LLM 应用程序中最重要的组件开始——LLM 本身！
### LLM 和聊天模型
大型语言模型及其聊天变体是基于 LLM 的应用程序的主要瓶颈。
大型语言模型可能需要**几秒钟**才能对查询生成完整的响应。这比应用程序对最终用户具有响应性的**约 200-300 毫秒**的阈值要慢得多。
使应用程序具有更高的响应性的关键策略是显示中间进度；即，逐个令牌流式传输模型的输出。
我们将展示使用聊天模型进行流式传输的示例。从以下选项中选择一个：

import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="model"
/>

让我们从同步 `stream` API 开始：
```python
chunks = []
for chunk in model.stream("天空是什么颜色？"):
    chunks.append(chunk)
    print(chunk.content, end="|", flush=True)
```
```output
天|空|是|什|么|颜|色|？|
```
或者，如果您在异步环境中工作，可以考虑使用异步 `astream` API：
```python
chunks = []
async for chunk in model.astream("天空是什么颜色？"):
    chunks.append(chunk)
    print(chunk.content, end="|", flush=True)
```
```output
天|空|是|什|么|颜|色|？|
```
让我们检查其中一个块：
```python
chunks[0]
```
```output
AIMessageChunk(content='天', id='run-b36bea64-5511-4d7a-b6a3-a07b3db0c8e7')
```
我们得到了一个称为 `AIMessageChunk` 的东西。该块表示 `AIMessage` 的一部分。
消息块是可叠加的——可以简单地将它们相加以获得到目前为止的响应状态！
```python
chunks[0] + chunks[1] + chunks[2] + chunks[3] + chunks[4]
```
```output
AIMessageChunk(content='天空是什么颜色', id='run-b36bea64-5511-4d7a-b6a3-a07b3db0c8e7')
```
### 链
几乎所有的 LLM 应用程序都涉及不止一步的操作，而不仅仅是调用语言模型。
让我们使用 `LangChain 表达式语言` (`LCEL`) 构建一个简单的链，该链结合了一个提示、模型和解析器，并验证流式传输是否正常工作。
我们将使用 [`StrOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.string.StrOutputParser.html) 来解析模型的输出。这是一个简单的解析器，从 `AIMessageChunk` 中提取 `content` 字段，给出模型返回的 `token`。
:::tip
LCEL 是一种*声明式*的方式，通过将不同的 LangChain 原语链接在一起来指定一个“程序”。使用 LCEL 创建的链可以自动实现 `stream` 和 `astream`，从而实现对最终输出的流式传输。事实上，使用 LCEL 创建的链实现了整个标准 Runnable 接口。
:::
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template("给我讲一个关于{topic}的笑话")
parser = StrOutputParser()
chain = prompt | model | parser
async for chunk in chain.astream({"topic": "鹦鹉"}):
    print(chunk, end="|", flush=True)
```
```output
这里有一个关于鹦鹉的笑话：
一个男人去宠物店买一只鹦鹉。店主给他看了两只羽毛华丽的鹦鹉。
“这里有一只会说话的鹦鹉和一只不会说话的鹦鹉，”店主说道。“会说话的鹦鹉卖100美元，而不会说话的鹦鹉只要20美元。”
男人说：“我要那只20美元的不会说话的鹦鹉。”
他付款带走了那只鹦鹉。当他走在街上时，鹦鹉抬头看着他说：“你知道吗，你真是个愚蠢的人！”
男人惊讶地看着鹦鹉，不敢相信。鹦鹉接着说：“是的，你被狠狠地骗了！我和那只会说话的鹦鹉一样会说话，而你只花了20美元买我！”
请注意，即使我们在上面的链条末尾使用了`parser`，我们仍然可以获得流式输出。`parser`会对每个流式块进行操作。许多[LCEL基元](/docs/how_to#langchain-expression-language-lcel)也支持这种转换式的流式传递，这在构建应用程序时非常方便。
自定义函数可以被设计为返回生成器，这样就能够操作流。
某些可运行实体，如[提示模板](/docs/how_to#prompt-templates)和[聊天模型](/docs/how_to#chat-models)，无法处理单个块，而是聚合所有先前的步骤。这些可运行实体可以中断流处理。
:::注意
LangChain表达语言允许您将链的构建与使用模式（例如同步/异步、批处理/流式等）分开。如果这与您构建的内容无关，您也可以依赖于标准的**命令式**编程方法，通过在每个组件上调用`invoke`、`batch`或`stream`，将结果分配给变量，然后根据需要在下游使用它们。
:::
### 使用输入流
如果您想要在输出生成时从中流式传输JSON，该怎么办呢？
如果您依赖`json.loads`来解析部分JSON，那么解析将失败，因为部分JSON不会是有效的JSON。
您可能会束手无策，声称无法流式传输JSON。
事实证明，有一种方法可以做到这一点——解析器需要在**输入流**上操作，并尝试将部分JSON“自动完成”为有效状态。
让我们看看这样一个解析器的运行，以了解这意味着什么。
```python
from langchain_core.output_parsers import JsonOutputParser
chain = (
    model | JsonOutputParser()
)  # 由于Langchain旧版本中的一个错误，JsonOutputParser未能从某些模型中流式传输结果
async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`"
):
    print(text, flush=True)
```
```output
{}
{'countries': []}
{'countries': [{}]}
{'countries': [{'name': ''}]}
{'countries': [{'name': 'France'}]}
{'countries': [{'name': 'France', 'population': 67}]}
{'countries': [{'name': 'France', 'population': 67413}]}
{'countries': [{'name': 'France', 'population': 67413000}]}
{'countries': [{'name': 'France', 'population': 67413000}, {}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': ''}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain'}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': ''}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan'}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125584}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125584000}]}
```
现在，让我们**中断**流式传输。我们将使用前面的示例，并在最后附加一个提取函数，从最终的JSON中提取国家名称。
:::警告
链中的任何操作**仅针对已完成的输入**而不是**输入流**的步骤，都可能通过`stream`或`astream`中断流功能。
:::
:::提示
稍后，我们将讨论`astream_events` API，该API将从中间步骤流式传输结果。即使链中包含仅操作**已完成输入**的步骤，此API也将从中间步骤流式传输结果。
::：
```
```python
from langchain_core.output_parsers import (
    JsonOutputParser,
)
# 一个在最终输入上操作的函数
# 而不是在输入流上操作
def _extract_country_names(inputs):
    """一个不在输入流上操作并且中断流的函数。"""
    if not isinstance(inputs, dict):
        return ""
    if "countries" not in inputs:
        return ""
    countries = inputs["countries"]
    if not isinstance(countries, list):
        return ""
    country_names = [
        country.get("name") for country in countries if isinstance(country, dict)
    ]
    return country_names
chain = model | JsonOutputParser() | _extract_country_names
async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`"
):
    print(text, end="|", flush=True)
```
```output
['France', 'Spain', 'Japan']|
```
#### 生成器函数
让我们使用可以在**输入流**上操作的生成器函数来修复流。
:::tip
生成器函数（使用`yield`的函数）允许编写可以在**输入流**上操作的代码。
:::
```python
from langchain_core.output_parsers import JsonOutputParser
async def _extract_country_names_streaming(input_stream):
    """一个在输入流上操作的函数。"""
    country_names_so_far = set()
    async for input in input_stream:
        if not isinstance(input, dict):
            continue
        if "countries" not in input:
            continue
        countries = input["countries"]
        if not isinstance(countries, list):
            continue
        for country in countries:
            name = country.get("name")
            if not name:
                continue
            if name not in country_names_so_far:
                yield name
                country_names_so_far.add(name)
chain = model | JsonOutputParser() | _extract_country_names_streaming
async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
):
    print(text, end="|", flush=True)
```
```output
France|Spain|Japan|
```
:::note
因为上面的代码依赖于 JSON 自动补全，你可能会看到国家名称的部分（例如 `Sp` 和 `Spain`），这不是我们希望得到的提取结果！
我们关注的是流概念，而不一定是链的结果。
:::
### 非流组件
一些内置组件，比如 Retrievers，并不提供任何`流`。如果我们尝试对它们进行`流`会发生什么？🤨
```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
template = """Answer the question based only on the following context:
{context}
Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho", "harrison likes spicy food"],
    embedding=OpenAIEmbeddings(),
)
retriever = vectorstore.as_retriever()
chunks = [chunk for chunk in retriever.stream("where did harrison work?")]
chunks
```
```output
[[Document(page_content='harrison worked at kensho'),
  Document(page_content='harrison likes spicy food')]]
```
流只产生了该组件的最终结果。
这是可以接受的🥹！并非所有组件都必须实现流 -- 在某些情况下，流要么是不必要的，要么很困难，或者根本没有意义。
:::tip
使用非流组件构建的 LCEL 链，在许多情况下仍然能够进行流处理，部分输出的流处理将从链中最后一个非流步骤之后开始。
:::
```python
retrieval_chain = (
    {
        "context": retriever.with_config(run_name="Docs"),
        "question": RunnablePassthrough(),
    }
    | prompt
    | model
    | StrOutputParser()
)
```
```python
for chunk in retrieval_chain.stream(
    "Where did harrison work? " "Write 3 made up sentences about this place."
):
    print(chunk, end="|", flush=True)
```
```output
Base|d on| the| given| context|,| Harrison| worke|d at| K|ens|ho|.|
Here| are| |3| |made| up| sentences| about| this| place|:|
1|.| K|ens|ho| was| a| cutting|-|edge| technology| company| known| for| its| innovative| solutions| in| artificial| intelligence| an|d data| analytics|.|
2|.| The| modern| office| space| at| K|ens|ho| feature|d open| floor| plans|,| collaborative| work|sp|aces|,| an|d a| vib|rant| atmosphere| that| fos|tere|d creativity| an|d team|work|.|
3|.| With| its| prime| location| in| the| heart| of| the| city|,| K|ens|ho| attracte|d top| talent| from| aroun|d the| worl|d,| creating| a| diverse| an|d dynamic| work| environment|.|
```
现在我们已经了解了`stream`和`astream`的工作原理，让我们进入事件流的世界。🏞️
## 使用事件流
事件流是一个**beta** API。这个API可能会根据反馈略微更改。
:::note
本指南演示了`V2` API，并且需要 langchain-core >= 0.2。对于与旧版本 LangChain 兼容的`V1` API，请参阅[这里](https://python.langchain.com/v0.1/docs/expression_language/streaming/#using-stream-events)。
:::
```python
import langchain_core
langchain_core.__version__
```
为了使`astream_events` API正常工作：
- 在代码中尽可能使用`async`（例如，异步工具等）
- 如果定义自定义函数/可运行项，请传播回调
- 在没有 LCEL 的情况下使用可运行项时，请确保在LLMs上调用`.astream()`而不是`.ainvoke`以强制LLM流式传输令牌
- 如果有任何不符合预期的情况，请告诉我们！ :)
### 事件参考
下面是一个参考表，显示各种可运行对象可能发出的一些事件。
:::note
当流式传输正确实现时，对于可运行项的输入直到输入流完全消耗后才会知道。这意味着`inputs`通常仅包括`end`事件，而不包括`start`事件。
:::
| 事件                | 名称             | 块                             | 输入                                         | 输出                                          |
|----------------------|------------------|---------------------------------|-----------------------------------------------|-------------------------------------------------|
| on_chat_model_start  | [模型名称]      |                                 | {"messages": [[SystemMessage, HumanMessage]]} |                                                 || on_chat_model_stream | [模型名称]      | AIMessageChunk(content="hello") |                                               |                                                 |
| on_chat_model_end    | [模型名称]      |                                 | {"messages": [[SystemMessage, HumanMessage]]} | AIMessageChunk(content="hello world")           |
| on_llm_start         | [模型名称]      |                                 | {'input': 'hello'}                            |                                                 |
| on_llm_stream        | [模型名称]      | 'Hello'                         |                                               |                                                 |
| on_llm_end           | [模型名称]      |                                 | 'Hello human!'                                |                                                 |
| on_chain_start       | format_docs      |                                 |                                               |                                                 |
| on_chain_stream      | format_docs      | "hello world!, goodbye world!"  |                                               |                                                 |
| on_chain_end         | format_docs      |                                 | [Document(...)]                               | "hello world!, goodbye world!"                  |
| on_tool_start        | some_tool        |                                 | {"x": 1, "y": "2"}                            |                                                 |
| on_tool_end          | some_tool        |                                 |                                               | {"x": 1, "y": "2"}                              |
| on_retriever_start   | [检索器名称]    |                                 | {"query": "hello"}                            |                                                 |
| on_retriever_end     | [检索器名称]    |                                 | {"query": "hello"}                            | [Document(...), ..]                             |
| on_prompt_start      | [模板名称]      |                                 | {"question": "hello"}                         |                                                 |
| on_prompt_end        | [模板名称]      |                                 | {"question": "hello"}                         | ChatPromptValue(messages: [SystemMessage, ...]) |
### 聊天模型
让我们首先看一下聊天模型产生的事件。
```python
events = []
async for event in model.astream_events("hello", version="v2"):
    events.append(event)
```
```output
/home/eugene/src/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: This API is in beta and may change in the future.
  warn_beta(
```
:::note
嘿，API中那个有趣的`version="v2"`参数是什么意思？😾
这是一个**beta API**，我们几乎肯定会对其进行一些更改（事实上，我们已经做了！）
这个版本参数将允许我们最小化对您代码的破坏性更改。
简而言之，我们现在让您感到烦恼，这样以后就不必再烦恼了。
`v2`仅适用于 langchain-core>=0.2.0。
:::
让我们看一下一些开始事件和一些结束事件。
```python
events[:3]
```
```output
[{'event': 'on_chat_model_start',
  'data': {'input': 'hello'},
  'name': 'ChatAnthropic',
  'tags': [],
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='Hello', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='!', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}}]
```
```python
events[-2:]
```
```output
[{'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='?', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}},
 {'event': 'on_chat_model_end',
  'data': {'output': AIMessageChunk(content='Hello! How can I assist you today?', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}}]
```
### 链
让我们重新查看一个示例链，该链解析了流式 JSON 以探索流事件 API。
```python
chain = (
    model | JsonOutputParser()
)  # 由于 Langchain 旧版本中的一个错误，JsonOutputParser 未能从某些模型中流式传输结果
events = [
    event
    async for event in chain.astream_events(
        "output a list of the countries france, spain and japan and their populations in JSON format. "
        'Use a dict with an outer key of "countries" which contains a list of countries. '
        "Each country should have the key `name` and `population`",
        version="v2",
    )
]
```
如果您查看前几个事件，您会注意到有 **3** 个不同的开始事件，而不是 **2** 个开始事件。
这三个开始事件对应于：
1. 链（模型 + 解析器）
2. 模型
3. 解析器
```python
events[:3]
```
```output
[{'event': 'on_chain_start',
  'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'},
  'name': 'RunnableSequence',
  'tags': [],
  'run_id': '4765006b-16e2-4b1d-a523-edd9fd64cb92',
  'metadata': {}},
 {'event': 'on_chat_model_start',
  'data': {'input': {'messages': [[HumanMessage(content='output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`')]]}},
  'name': 'ChatAnthropic',
  'tags': ['seq:step:1'],
  'run_id': '0320c234-7b52-4a14-ae4e-5f100949e589',
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='{', id='run-0320c234-7b52-4a14-ae4e-5f100949e589')},
  'run_id': '0320c234-7b52-4a14-ae4e-5f100949e589',
  'name': 'ChatAnthropic',
  'tags': ['seq:step:1'],
  'metadata': {}}]
```
如果您查看最后 3 个事件，您认为会看到什么？中间呢？
让我们使用此 API 输出模型和解析器的流事件。我们忽略链的开始事件、结束事件和事件。
```python
num_events = 0
async for event in chain.astream_events(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
    version="v2",
):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(
            f"Chat model chunk: {repr(event['data']['chunk'].content)}",
            flush=True,
        )
    if kind == "on_parser_stream":
        print(f"Parser chunk: {event['data']['chunk']}", flush=True)
    num_events += 1
    if num_events > 30:
        # 截断输出
        print("...")
        break
```
```output
Chat model chunk: '{'
Parser chunk: {}
Chat model chunk: '\n  '
Chat model chunk: '"'
Chat model chunk: 'countries'
Chat model chunk: '":'
Chat model chunk: ' ['
Parser chunk: {'countries': []}
Chat model chunk: '\n    '
Chat model chunk: '{'
Parser chunk: {'countries': [{}]}
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'name'
Chat model chunk: '":'
Chat model chunk: ' "'
Parser chunk: {'countries': [{'name': ''}]}
Chat model chunk: 'France'
Parser chunk: {'countries': [{'name': 'France'}]}
Chat model chunk: '",'
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'population'
...```
```
由于模型和解析器都支持流式处理，我们可以实时看到来自这两个组件的流事件！是不是很酷？🦜
### 过滤事件
由于此 API 生成了如此多的事件，能够对事件进行过滤非常有用。
您可以按组件的 `name`、组件的 `tags` 或组件的 `type` 进行过滤。
#### 按名称
```python
chain = model.with_config({"run_name": "model"}) | JsonOutputParser().with_config(
    {"run_name": "my_parser"}
)
max_events = 0
async for event in chain.astream_events(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
    version="v2",
    include_names=["my_parser"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # 截断输出
        print("...")
        break
```
```output
{'event': 'on_parser_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'my_parser', 'tags': ['seq:step:2'], 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': []}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': ''}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France'}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}, {}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}, {'name': ''}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
...
```
#### 按类型
```python
chain = model.with_config({"run_name": "model"}) | JsonOutputParser().with_config(
    {"run_name": "my_parser"}
)
max_events = 0
async for event in chain.astream_events(
    'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`',
    version="v2",
    include_types=["chat_model"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # 截断输出
        print("...")
        break
```
```output
{'event': 'on_chat_model_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'model', 'tags': ['seq:step:1'], 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n  ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
```
```python
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='countries', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='":', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' [', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n    ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n      ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
...
```
#### 按标签分类
:::caution
标签会被给定可运行项的子组件继承。
如果您正在使用标签进行过滤，请确保这正是您想要的。
:::
```python
chain = (model | JsonOutputParser()).with_config({"tags": ["my_chain"]})
max_events = 0
async for event in chain.astream_events(
    'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`',
    version="v2",
    include_tags=["my_chain"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # 截断输出
        print("...")
        break
```
```output
{'event': 'on_chain_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'RunnableSequence', 'tags': ['my_chain'], 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'metadata': {}}
{'event': 'on_chat_model_start', 'data': {'input': {'messages': [[HumanMessage(content='output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`')]]}}, 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_parser_start', 'data': {}, 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {}}, 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': {}}, 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'name': 'RunnableSequence', 'tags': ['my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n  ', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-efd3c8af-4be5-4f-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
```
```python
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='国家', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='":', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' [', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
...
```
### 非流式组件
记得有些组件不适合流式处理，因为它们不操作**输入流**吗？
虽然这些组件可能会破坏使用`astream`时最终输出的流式处理，但`astream_events`仍会从支持流式处理的中间步骤产生流式事件！
```python
# 不支持流式处理的函数。
# 它操作的是最终的输入，而不是
# 操作输入流。
def _extract_country_names(inputs):
    """一个不操作输入流且会破坏流式处理的函数。"""
    if not isinstance(inputs, dict):
        return ""
    if "countries" not in inputs:
        return ""
    countries = inputs["countries"]
    if not isinstance(countries, list):
        return ""
    country_names = [
        country.get("name") for country in countries if isinstance(country, dict)
    ]
    return country_names
chain = (
    model | JsonOutputParser() | _extract_country_names
)  # 这个解析器目前只适用于 OpenAI
```
正如预期的那样，`astream` API无法正确工作，因为`_extract_country_names`不操作流。
```python
async for chunk in chain.astream(
    "以 JSON 格式输出法国、西班牙和日本及其人口的国家列表。"
    '使用一个包含国家列表的外键为“countries”的字典。'
    "每个国家应该有“name”和“population”键",
):
    print(chunk, flush=True)
```
```output
['法国', '西班牙', '日本']
```
现在，让我们通过`astream_events`确认，我们仍然可以看到来自模型和解析器的流式输出。
```python
num_events = 0
async for event in chain.astream_events(
    "以 JSON 格式输出法国、西班牙和日本及其人口的国家列表。"
    '使用一个包含国家列表的外键为“countries”的字典。'
    "每个国家应该有“name”和“population”键",
    version="v2",
):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(
            f"聊天模型块：{repr(event['data']['chunk'].content)}",
            flush=True,
        )
    if kind == "on_parser_stream":
        print(f"解析器块：{event['data']['chunk']}", flush=True)
    num_events += 1
    if num_events > 30:
        # 截断输出
        print("...")
        break
```
```output
聊天模型块：'{'
解析器块：{}
聊天模型块：'\n  '
聊天模型块：'"'
聊天模型块：'countries'
聊天模型块：'":'
聊天模型块：' ['
解析器块：{'countries': []}
聊天模型块：'\n    '
聊天模型块：'{'
解析器块：{'countries': [{}]}
聊天模型块：'\n      '
聊天模型块：'"'
聊天模型块：'name'
聊天模型块：'":'
聊天模型块：' "'
解析器块：{'countries': [{'name': ''}]}
聊天模型块：'法国'
解析器块：{'countries': [{'name': '法国'}]}
聊天模型块：'",'
聊天模型块：'\n      '
聊天模型块：'"'
聊天模型块：'population'
聊天模型块：'":'
聊天模型块：' '
聊天模型块：'67'
解析器块：{'countries': [{'name': '法国', 'population': 67}]}
...
```
### 传播回调
:::caution
如果在工具中调用运行时，您需要将回调传播给运行时；否则，将不会生成任何流事件。
:::
:::note
当使用`RunnableLambdas`或`@chain`装饰器时，回调会在幕后自动传播。
:::
```python
from langchain_core.runnables import RunnableLambda
from langchain_core.tools import tool
def reverse_word(word: str):
    return word[::-1]
reverse_word = RunnableLambda(reverse_word)
@tool
def bad_tool(word: str):
    """不传播回调的自定义工具。"""
    return reverse_word.invoke(word)
async for event in bad_tool.astream_events("hello", version="v2"):
    print(event)
```
```output
{'event': 'on_tool_start', 'data': {'input': 'hello'}, 'name': 'bad_tool', 'tags': [], 'run_id': 'ea900472-a8f7-425d-b627-facdef936ee8', 'metadata': {}}
```
```json
{'event': 'on_chain_start', 'data': {'input': 'hello'}, 'name': 'reverse_word', 'tags': [], 'run_id': '77b01284-0515-48f4-8d7c-eb27c1882f86', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': 'olleh', 'input': 'hello'}, 'run_id': '77b01284-0515-48f4-8d7c-eb27c1882f86', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_tool_end', 'data': {'output': 'olleh'}, 'run_id': 'ea900472-a8f7-425d-b627-facdef936ee8', 'name': 'bad_tool', 'tags': [], 'metadata': {}}
```
这里是一个正确传递回调的重新实现。现在我们注意到我们也从`reverse_word`可运行中获取事件。
```python
@tool
def correct_tool(word: str, callbacks):
    """一个能正确传递回调的工具。"""
    return reverse_word.invoke(word, {"callbacks": callbacks})
async for event in correct_tool.astream_events("hello", version="v2"):
    print(event)
```
```output
{'event': 'on_tool_start', 'data': {'input': 'hello'}, 'name': 'correct_tool', 'tags': [], 'run_id': 'd5ea83b9-9278-49cc-9f1d-aa302d671040', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': 'hello'}, 'name': 'reverse_word', 'tags': [], 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': 'olleh', 'input': 'hello'}, 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_tool_end', 'data': {'output': 'olleh'}, 'run_id': 'd5ea83b9-9278-49cc-9f1d-aa302d671040', 'name': 'correct_tool', 'tags': [], 'metadata': {}}
```
如果您在Runnable Lambdas或`@chains`中调用可运行的内容，那么回调将自动传递给您。
```python
from langchain_core.runnables import RunnableLambda
async def reverse_and_double(word: str):
    return await reverse_word.ainvoke(word) * 2
reverse_and_double = RunnableLambda(reverse_and_double)
await reverse_and_double.ainvoke("1234")
async for event in reverse_and_double.astream_events("1234", version="v2"):
    print(event)
```
```output
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_and_double', 'tags': [], 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_word', 'tags': [], 'run_id': '5cf26fc8-840b-4642-98ed-623dda28707a', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '4321', 'input': '1234'}, 'run_id': '5cf26fc8-840b-4642-98ed-623dda28707a', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': '43214321'}, 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '43214321'}, 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
```
使用`@chain`装饰器：
```python
from langchain_core.runnables import chain
@chain
async def reverse_and_double(word: str):
    return await reverse_word.ainvoke(word) * 2
await reverse_and_double.ainvoke("1234")
async for event in reverse_and_double.astream_events("1234", version="v2"):
    print(event)
```
```output
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_and_double', 'tags': [], 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_word', 'tags': [], 'run_id': '64fc99f0-5d7d-442b-b4f5-4537129f67d1', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '4321', 'input': '1234'}, 'run_id': '64fc99f0-5d7d-442b-b4f5-4537129f67d1', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': '43214321'}, 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '43214321'}, 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
```
## 下一步
现在您已经学会了如何在LangChain中流式传输最终输出和内部步骤。
要了解更多信息，请查看本部分中的其他操作指南，或查看[Langchain表达语言的概念指南](/docs/concepts/#langchain-expression-language/)。 
