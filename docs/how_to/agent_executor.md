# 构建一个代理

单独来说，语言模型无法采取行动 - 它们只能输出文本。

LangChain 的一个重要用例是创建**代理**。

代理是使用 LLM 作为推理引擎的系统，用于确定应采取哪些行动以及这些行动的输入应该是什么。

然后可以将这些行动的结果反馈给代理，并确定是否需要更多行动，或者是否可以结束。

在本教程中，我们将构建一个可以与多种不同工具进行交互的代理：一个是本地数据库，另一个是搜索引擎。您将能够向该代理提问，观察它调用工具，并与它进行对话。

:::important

本节将介绍使用 LangChain 代理进行构建。LangChain 代理适合入门，但在一定程度之后，您可能希望拥有它们无法提供的灵活性和控制性。要使用更高级的代理，我们建议查看[LangGraph](/docs/concepts/#langgraph)。

:::

## 概念

我们将涵盖的概念包括：

- 使用[语言模型](/docs/concepts/#chat-models)，特别是它们的工具调用能力

- 创建[检索器](/docs/concepts/#retrievers)以向我们的代理公开特定信息

- 使用搜索[工具](/docs/concepts/#tools)在线查找信息

- [`聊天历史`](/docs/concepts/#chat-history)，允许聊天机器人“记住”过去的交互，并在回答后续问题时考虑它们。

- 使用[LangSmith](/docs/concepts/#langsmith)调试和跟踪您的应用程序

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用[Jupyter 笔记本](https://jupyter.org/)，并假定读者也在使用。Jupyter 笔记本非常适合学习如何使用 LLM 系统，因为通常会出现问题（意外输出、API 崩溃等），在交互式环境中阅读指南是更好地理解它们的好方法。

这些教程和其他教程可能最方便在 Jupyter 笔记本中运行。请参阅[此处](https://jupyter.org/install)了解安装说明。

### 安装

要安装 LangChain，请运行：

```bash
pip install langchain
```

或者使用 Conda：

```bash
conda install langchain -c conda-forge
```

有关更多详细信息，请参阅我们的[安装指南](/docs/how_to/installation)。

### LangSmith

使用 LangChain 构建的许多应用程序将包含多个步骤，其中会多次调用 LLM。

随着这些应用程序变得越来越复杂，能够检查链或代理内部发生了什么变得至关重要。

这样做的最佳方式是使用[LangSmith](https://smith.langchain.com)。

在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，在笔记本中，您可以使用以下方式设置：

```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 定义工具

我们首先需要创建我们想要使用的工具。我们将使用两个工具：[Tavily](/docs/integrations/tools/tavily_search)（用于在线搜索），然后是我们将创建的本地索引上的检索器。

### [Tavily](/docs/integrations/tools/tavily_search)

LangChain 中有一个内置工具，可以轻松使用 Tavily 搜索引擎作为工具。

请注意，这需要一个 API 密钥 - 他们有一个免费的层级，但如果您没有或不想创建一个，您可以忽略这一步。

创建 API 密钥后，您需要将其导出为：

```bash
export TAVILY_API_KEY="..."
```
```python
from langchain_community.tools.tavily_search import TavilySearchResults
```
```python
search = TavilySearchResults(max_results=2)
```
```python
search.invoke("what is the weather in SF")
```
```output
[{'url': 'https://www.weatherapi.com/',
  'content': "{'location': {'name': 'San Francisco', 'region': 'California', 'country': 'United States of America', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1714000492, 'localtime': '2024-04-24 16:14'}, 'current': {'last_updated_epoch': 1713999600, 'last_updated': '2024-04-24 16:00', 'temp_c': 15.6, 'temp_f': 60.1, 'is_day': 1, 'condition': {'text': 'Overcast', 'icon': '//cdn.weatherapi.com/weather/64x64/day/122.png', 'code': 1009}, 'wind_mph': 10.5, 'wind_kph': 16.9, 'wind_degree': 330, 'wind_dir': 'NNW', 'pressure_mb': 1018.0, 'pressure_in': 30.06, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 72, 'cloud': 100, 'feelslike_c': 15.6, 'feelslike_f': 60.1, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 14.8, 'gust_kph': 23.8}}"},
 {'url': 'https://www.weathertab.com/en/c/e/04/united-states/california/san-francisco/',
  'content': 'San Francisco Weather Forecast for Apr 2024 - Risk of Rain Graph. Rain Risk Graph: Monthly Overview. Bar heights indicate rain risk percentages. Yellow bars mark low-risk days, while black and grey bars signal higher risks. Grey-yellow bars act as buffers, advising to keep at least one day clear from the riskier grey and black days, guiding ...'}]
```

### 召回器

我们还将在自己的一些数据上创建一个召回器。有关每个步骤的更深入解释，请参阅[此教程](/docs/tutorials/rag)。

```python
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
loader = WebBaseLoader("https://docs.smith.langchain.com/overview")
docs = loader.load()
documents = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200
).split_documents(docs)
vector = FAISS.from_documents(documents, OpenAIEmbeddings())
retriever = vector.as_retriever()
```
```python
retriever.invoke("how to upload a dataset")[0]
```
```output
Document(page_content='# The data to predict and grade over    evaluators=[exact_match], # The evaluators to score the results    experiment_prefix="sample-experiment", # The name of the experiment    metadata={      "version": "1.0.0",      "revision_id": "beta"    },)import { Client, Run, Example } from \'langsmith\';import { runOnDataset } from \'langchain/smith\';import { EvaluationResult } from \'langsmith/evaluation\';const client = new Client();// Define dataset: these are your test casesconst datasetName = "Sample Dataset";const dataset = await client.createDataset(datasetName, {    description: "A sample dataset in LangSmith."});await client.createExamples({    inputs: [        { postfix: "to LangSmith" },        { postfix: "to Evaluations in LangSmith" },    ],    outputs: [        { output: "Welcome to LangSmith" },        { output: "Welcome to Evaluations in LangSmith" },    ],    datasetId: dataset.id,});// Define your evaluatorconst exactMatch = async ({ run, example }: { run: Run; example?:', metadata={'source': 'https://docs.smith.langchain.com/overview', 'title': 'Getting started with LangSmith | \uf8ffü¶úÔ∏è\uf8ffüõ†Ô∏è LangSmith', 'description': 'Introduction', 'language': 'en'})
```

现在，我们已经填充了我们将要进行召回的索引，我们可以轻松地将其转换为一个工具（代理程序正确使用所需的格式）。

```python
from langchain.tools.retriever import create_retriever_tool
```
```python
retriever_tool = create_retriever_tool(
    retriever,
    "langsmith_search",
    "Search for information about LangSmith. For any questions about LangSmith, you must use this tool!",
)
```

### 工具

既然我们都创建好了，我们可以创建一个工具列表，以便在下游使用。

```python
tools = [search, retriever_tool]
```

## 使用语言模型

接下来，让我们学习如何使用语言模型来调用工具。LangChain支持许多可以互换使用的不同语言模型 - 选择您想要使用的语言模型！

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs openaiParams={`model="gpt-4"`} />
```

您可以通过传入消息列表来调用语言模型。默认情况下，响应是一个`content`字符串。

```python
from langchain_core.messages import HumanMessage
response = model.invoke([HumanMessage(content="hi!")])
response.content
```
```output
'Hello! How can I assist you today?'
```

现在，我们可以看看如何使这个模型能够调用工具。为了使其具备这种能力，我们使用`.bind_tools`来让语言模型了解这些工具。

```python
model_with_tools = model.bind_tools(tools)
```

现在我们可以调用模型了。让我们首先用一个普通的消息来调用它，看看它的响应。我们可以查看`content`字段和`tool_calls`字段。

```python
response = model_with_tools.invoke([HumanMessage(content="Hi!")])
print(f"ContentString: {response.content}")
print(f"ToolCalls: {response.tool_calls}")
```
```output
ContentString: Hello! How can I assist you today?
ToolCalls: []
```

现在，让我们尝试使用一些期望调用工具的输入来调用它。

```python
response = model_with_tools.invoke([HumanMessage(content="What's the weather in SF?")])
print(f"ContentString: {response.content}")
print(f"ToolCalls: {response.tool_calls}")
```
```output
ContentString: 
ToolCalls: [{'name': 'tavily_search_results_json', 'args': {'query': 'current weather in San Francisco'}, 'id': 'call_4HteVahXkRAkWjp6dGXryKZX'}]
```

我们可以看到现在没有内容，但有一个工具调用！它要求我们调用Tavily Search工具。

这并不是在调用该工具 - 它只是告诉我们要调用。为了实际调用它，我们将创建我们的代理程序。

## 创建代理程序

既然我们已经定义了工具和LLM，我们可以创建代理程序。我们将使用一个工具调用代理程序 - 有关此类代理程序以及其他选项的更多信息，请参阅[此指南](/docs/concepts/#agent_types/)。

我们可以首先选择要用来指导代理程序的提示。

如果您想查看此提示的内容并访问LangSmith，您可以转到：

[https://smith.langchain.com/hub/hwchase17/openai-functions-agent](https://smith.langchain.com/hub/hwchase17/openai-functions-agent)

```python
from langchain import hub
# 获取要使用的提示 - 您可以修改这个！
prompt = hub.pull("hwchase17/openai-functions-agent")
prompt.messages
```
```output
[SystemMessagePromptTemplate(prompt=PromptTemplate(input_variables=[], template='You are a helpful assistant')),
 MessagesPlaceholder(variable_name='chat_history', optional=True),
 HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['input'], template='{input}')),
 MessagesPlaceholder(variable_name='agent_scratchpad')]
```

现在，我们可以使用LLM、提示和工具初始化代理。代理负责接收输入并决定采取什么行动。关键的是，代理不执行这些操作 - 这是由AgentExecutor（下一步）完成的。有关如何考虑这些组件的更多信息，请参阅我们的[概念指南](/docs/concepts/#agents)。

请注意，我们传递的是`model`，而不是`model_with_tools`。这是因为`create_tool_calling_agent`会在幕后调用`.bind_tools`。

```python
from langchain.agents import create_tool_calling_agent
agent = create_tool_calling_agent(model, tools, prompt)
```

最后，我们将代理（大脑）与AgentExecutor中的工具结合起来（AgentExecutor将重复调用代理并执行工具）。

```python
from langchain.agents import AgentExecutor
agent_executor = AgentExecutor(agent=agent, tools=tools)
```

## 运行代理

现在，我们可以在几个查询上运行代理！请注意，目前这些都是**无状态**查询（它不会记住先前的交互）。

首先，让我们看看当不需要调用工具时它如何回应：

```python
agent_executor.invoke({"input": "hi!"})
```
```output
{'input': 'hi!', 'output': 'Hello! How can I assist you today?'}
```

为了确切了解底层发生了什么（并确保它没有调用工具），我们可以查看[LangSmith跟踪](https://smith.langchain.com/public/8441812b-94ce-4832-93ec-e1114214553a/r)

现在让我们尝试一个应该调用检索器的示例：

```python
agent_executor.invoke({"input": "how can langsmith help with testing?"})
```
```output
{'input': 'how can langsmith help with testing?',
 'output': 'LangSmith是一个帮助构建生产级语言学习模型（LLM）应用程序的平台。它可以通过以下几种方式帮助测试：\n\n1. **监控和评估**：LangSmith允许密切监视和评估您的应用程序。这有助于确保您的应用程序的质量并自信地部署它。\n\n2. **跟踪**：LangSmith具有跟踪功能，可有助于调试和了解您的应用程序的行为。\n\n3. **评估功能**：LangSmith具有用于评估LLM性能的内置工具。\n\n4. **提示中心**：这是内置在LangSmith中的提示管理工具，可帮助测试不同提示及其响应。\n\n请注意，要使用LangSmith，您需要安装它并创建API密钥。该平台提供Python和Typescript SDK供使用。它独立运行，不需要使用LangChain。'}
```

让我们查看[LangSmith跟踪](https://smith.langchain.com/public/762153f6-14d4-4c98-8659-82650f860c62/r)以确保它实际上在调用该工具。

现在让我们尝试一个需要调用搜索工具的示例：

```python
agent_executor.invoke({"input": "whats the weather in sf?"})
```
```output
{'input': 'whats the weather in sf?',
 'output': '旧金山目前的天气是部分多云，温度为16.1°C（61.0°F）。风来自西北偏西，速度为10.5英里/小时。湿度为67%。[来源](https://www.weatherapi.com/)'}
```

我们可以查看[LangSmith跟踪](https://smith.langchain.com/public/36df5b1a-9a0b-4185-bae2-964e1d53c665/r)以确保它有效地调用了搜索工具。

## 添加记忆

如前所述，此代理是无状态的。这意味着它不会记住先前的交互。要给它记忆，我们需要传递先前的`chat_history`。注意：由于我们使用的提示，它需要被称为`chat_history`。如果我们使用不同的提示，我们可以更改变量名

```python
# 这里我们为chat_history传入了一个空消息列表，因为这是对话中的第一条消息
agent_executor.invoke({"input": "hi! my name is bob", "chat_history": []})
```
```output
{'input': 'hi! my name is bob',
 'chat_history': [],
 'output': '你好Bob！我今天能帮你什么？'}
```
```python
from langchain_core.messages import AIMessage, HumanMessage
```
```python
agent_executor.invoke(
    {
        "chat_history": [
            HumanMessage(content="hi! my name is bob"),
            AIMessage(content="你好Bob！我今天能帮你什么？"),
        ],
        "input": "我的名字是什么?",
    }
)
如果我们想要自动跟踪这些消息，我们可以将其包装在一个RunnableWithMessageHistory中。有关如何使用它的更多信息，请参见[此指南](/docs/how_to/message_history)。
```python

from langchain_community.chat_message_histories import ChatMessageHistory

from langchain_core.chat_history import BaseChatMessageHistory

from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}

def get_session_history(session_id: str) -> BaseChatMessageHistory:

    if session_id not in store:

        store[session_id] = ChatMessageHistory()

    return store[session_id]

```
因为我们有多个输入，我们需要指定两个事项：
- `input_messages_key`：用于将输入添加到对话历史记录中的键。
- `history_messages_key`：用于将加载的消息添加到其中的键。
```python
agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```
```python
agent_with_chat_history.invoke(
    {"input": "hi! I'm bob"},
    config={"configurable": {"session_id": "<foo>"}},
)
```
```output

{'input': "hi! I'm bob",

 'chat_history': [],

 'output': 'Hello Bob! How can I assist you today?'}

```
```python
agent_with_chat_history.invoke(
    {"input": "what's my name?"},
    config={"configurable": {"session_id": "<foo>"}},
)
```
```output

{'input': "what's my name?",

 'chat_history': [HumanMessage(content="hi! I'm bob"),

  AIMessage(content='Hello Bob! How can I assist you today?')],

 'output': 'Your name is Bob.'}

```

LangSmith示例跟踪：[https://smith.langchain.com/public/98c8d162-60ae-4493-aa9f-992d87bd0429/r](https://smith.langchain.com/public/98c8d162-60ae-4493-aa9f-992d87bd0429/r)

## 结论

这就是全部内容！在这个快速入门中，我们介绍了如何创建一个简单的代理。代理是一个复杂的主题，有很多东西可以学习！

:::important

本节介绍了如何使用LangChain代理进行构建。LangChain代理适用于入门，但在某个阶段之后，您可能希望获得它们无法提供的灵活性和控制性。如果要使用更高级的代理，请查看[LangGraph](/docs/concepts/#langgraph)。

:::

如果您想继续使用LangChain代理，以下是一些不错的高级指南：

- [如何使用LangGraph的内置版本的`AgentExecutor`](/docs/how_to/migrate_agent)

- [如何创建自定义代理](https://python.langchain.com/v0.1/docs/modules/agents/how_to/custom_agent/)

- [如何从代理中流式传输响应](https://python.langchain.com/v0.1/docs/modules/agents/how_to/streaming/)

- [如何从代理返回结构化输出](https://python.langchain.com/v0.1/docs/modules/agents/how_to/agent_structured/)