---
sidebar_position: 4
---
# 构建一个Agent
单独来看，语言模型无法采取行动 - 它们只能输出文本。
LangChain的一个重要用途是创建**代理**。
代理是使用LLM作为推理引擎的系统，用于确定要采取的行动以及这些行动的输入应该是什么。
然后，可以将这些行动的结果反馈给代理，并确定是否需要更多行动，或者是否可以结束。
在本教程中，我们将构建一个代理，该代理可以与多个不同的工具进行交互：一个是本地数据库，另一个是搜索引擎。您将能够向该代理提问，观察它调用工具，并与它进行对话。
## 概念
我们将涵盖的概念包括：
- 使用[语言模型](/docs/concepts/#chat-models)，特别是它们的工具调用能力
- 创建[检索器](/docs/concepts/#retrievers)以向代理公开特定信息
- 使用搜索[工具](/docs/concepts/#tools)在线查找信息
- 使用[LangGraph代理](/docs/concepts/#agents)，它使用LLM来思考要做什么，然后执行相应的操作
- 使用[LangSmith](/docs/concepts/#langsmith)调试和跟踪应用程序
## 设置
### Jupyter Notebook
本指南（以及文档中的大多数其他指南）使用[Jupyter笔记本](https://jupyter.org/)，并假设读者也在使用。Jupyter笔记本非常适合学习如何使用LLM系统，因为往往会出现问题（意外输出，API关闭等），在交互环境中阅读指南是更好地理解它们的好方法。
这个和其他教程可能最方便在Jupyter笔记本中运行。请参阅[这里](https://jupyter.org/install)以获取安装说明。
### 安装
要安装LangChain，请运行以下命令：

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from "@theme/CodeBlock";
<Tabs>
  <TabItem value="pip" label="Pip" default>
    <CodeBlock language="bash">pip install langchain</CodeBlock>
  </TabItem>
  <TabItem value="conda" label="Conda">
    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>
  </TabItem>
</Tabs>

有关更多详细信息，请参阅我们的[安装指南](/docs/how_to/installation)。

### LangSmith
使用LangChain构建的许多应用程序将包含多个步骤，其中包含多个LLM调用的调用。
随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。
这样做的最佳方法是使用[LangSmith](https://smith.langchain.com)。
在上面的链接上注册后，请确保设置您的环境变量以开始记录跟踪：
```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```
或者，如果在笔记本中，可以使用以下代码进行设置：
```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
## 定义工具
我们首先需要创建要使用的工具。我们将使用两个工具：[Tavily](/docs/integrations/tools/tavily_search)（用于在线搜索）和一个在本地索引上创建的检索器。
### [Tavily](/docs/integrations/tools/tavily_search)
LangChain中有一个内置工具，可以轻松使用Tavily搜索引擎作为工具。
请注意，这需要一个API密钥 - 它们有一个免费的层次，但如果您没有或不想创建一个，您可以忽略此步骤。
创建API密钥后，您需要将其导出为：
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
[{'url': 'https://weather.com/weather/tenday/l/San Francisco CA USCA0987:1:US',
  'content': "Comfy & Cozy\nThat's Not What Was Expected\nOutside\n'No-Name Storms' In Florida\nGifts From On High\nWhat To Do For Wheezing\nSurviving The Season\nStay Safe\nAir Quality Index\nAir quality is considered satisfactory, and air pollution poses little or no risk.\n Health & Activities\nSeasonal Allergies and Pollen Count Forecast\nNo pollen detected in your area\nCold & Flu Forecast\nFlu risk is low in your area\nWe recognize our responsibility to use data and technology for good. recents\nSpecialty Forecasts\n10 Day Weather-San Francisco, CA\nToday\nMon 18 | Day\nConsiderable cloudiness. Tue 19\nTue 19 | Day\nLight rain early...then remaining cloudy with showers in the afternoon. Wed 27\nWed 27 | Day\nOvercast with rain showers at times."},
 {'url': 'https://www.accuweather.com/en/us/san-francisco/94103/hourly-weather-forecast/347629',
  'content': 'Hourly weather forecast in San Francisco, CA. Check current conditions in San Francisco, CA with radar, hourly, and more.'}]
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
Document(page_content='import Clientfrom langsmith.evaluation = evaluateclient = Client()# Define dataset: these are your test casesdataset_name = "Sample Dataset"dataset = client.create_dataset(dataset_name, description="A sample dataset in LangSmith.")client.create_examples(    inputs=[        {"postfix": "to LangSmith"},        {"postfix": "to Evaluations in LangSmith"},    ],    outputs=[        {"output": "Welcome to LangSmith"},        {"output": "Welcome to Evaluations in LangSmith"},    ],    dataset_id=dataset.id,)# Define your evaluatordef exact_match(run, example):    return {"score": run.outputs["output"] == example.outputs["output"]}experiment_results = evaluate(    lambda input: "Welcome " + input[\'postfix\'], # Your AI system goes here    data=dataset_name, # The data to predict and grade over    evaluators=[exact_match], # The evaluators to score the results    experiment_prefix="sample-experiment", # The name of the experiment    metadata={      "version": "1.0.0",      "revision_id":', metadata={'source': 'https://docs.smith.langchain.com/overview', 'title': 'Getting started with LangSmith | 🦜️🛠️ LangSmith', 'description': 'Introduction', 'language': 'en'})
```
现在我们已经填充了我们将要进行召回的索引，我们可以轻松地将其转换为一个工具（代理程序正确使用所需的格式）。
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
既然我们已经创建了这两个，我们可以创建一个我们将在下游使用的工具列表。
```python
tools = [search, retriever_tool]
```
## 使用语言模型
接下来，让我们学习如何使用语言模型来调用工具。LangChain支持许多可以互换使用的不同语言模型 - 选择您想要使用的模型！
```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs openaiParams={`model="gpt-4"`} />
```
您可以通过传入一系列消息来调用语言模型。默认情况下，响应是一个`content`字符串。
```python
from langchain_core.messages import HumanMessage
response = model.invoke([HumanMessage(content="hi!")])
response.content
```
```output
'Hello! How can I assist you today?'
```
现在我们可以看看如何使这个模型能够调用工具。为了实现这一点，我们使用`.bind_tools`来让语言模型了解这些工具。
```python
model_with_tools = model.bind_tools(tools)
```
我们现在可以调用模型。让我们首先用一个普通消息来调用它，看看它如何响应。我们可以查看`content`字段和`tool_calls`字段。
```python
response = model_with_tools.invoke([HumanMessage(content="Hi!")])
print(f"ContentString: {response.content}")
print(f"ToolCalls: {response.tool_calls}")
```
```output
ContentString: Hello! How can I assist you today?
ToolCalls: []
```
现在，让我们尝试用一些期望调用工具的输入来调用它。
```python
response = model_with_tools.invoke([HumanMessage(content="What's the weather in SF?")])
print(f"ContentString: {response.content}")
print(f"ToolCalls: {response.tool_calls}")
```
```output
ContentString: 
ToolCalls: [{'name': 'tavily_search_results_json', 'args': {'query': 'current weather in SF'}, 'id': 'call_nfE1XbCqZ8eJsB8rNdn4MQZQ'}]
```
我们可以看到现在没有内容，但有一个工具调用！它要求我们调用Tavily Search工具。
这并不是在调用该工具 - 它只是告诉我们要调用。为了真正调用它，我们将创建我们的代理程序。
## 创建代理程序
现在我们已经定义了工具和LLM，我们可以创建代理程序。我们将使用[LangGraph](/docs/concepts/#langgraph)来构建代理程序。
目前我们正在使用高级接口来构建代理程序，但LangGraph的好处是，这个高级接口由一个低级、高度可控的API支持，以防您想要修改代理程序逻辑。
现在，我们可以使用LLM和工具来初始化代理程序。
请注意，我们传入的是`model`，而不是`model_with_tools`。这是因为`create_tool_calling_executor`将在幕后为我们调用`.bind_tools`。
```python
from langgraph.prebuilt import chat_agent_executor
agent_executor = chat_agent_executor.create_tool_calling_executor(model, tools)
```
## 运行代理
现在我们可以对几个查询运行代理了！请注意，目前这些都是**无状态**查询（它不会记住先前的交互）。请注意，代理将在交互结束时返回**最终**状态（其中包括任何输入，稍后我们将看到如何仅获取输出）。
首先，让我们看看当不需要调用工具时它如何响应：
```python
response = agent_executor.invoke({"messages": [HumanMessage(content="hi!")]})
response["messages"]
```
```output
[HumanMessage(content='hi!', id='1535b889-10a5-45d0-a1e1-dd2e60d4bc04'),
 AIMessage(content='Hello! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 129, 'total_tokens': 139}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-2c94c074-bdc9-4f01-8fd7-71cfc4777d55-0')]
```
为了确切了解内部发生了什么（并确保它没有调用工具），我们可以查看[LangSmith跟踪](https://smith.langchain.com/public/28311faa-e135-4d6a-ab6b-caecf6482aaa/r)
现在让我们在一个应该调用检索器的示例上尝试一下：
```python
response = agent_executor.invoke(
    {"messages": [HumanMessage(content="how can langsmith help with testing?")]}
)
response["messages"]
```
```output
[HumanMessage(content='how can langsmith help with testing?', id='04f4fe8f-391a-427c-88af-1fa064db304c'),
 AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_FNIgdO97wo51sKx3XZOGLHqT', 'function': {'arguments': '{\n  "query": "how can LangSmith help with testing"\n}', 'name': 'langsmith_search'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 22, 'prompt_tokens': 135, 'total_tokens': 157}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-51f6ea92-84e1-43a5-b1f2-bc0c12d8613f-0', tool_calls=[{'name': 'langsmith_search', 'args': {'query': 'how can LangSmith help with testing'}, 'id': 'call_FNIgdO97wo51sKx3XZOGLHqT'}]),
 ToolMessage(content="Getting started with LangSmith | 🦜️🛠️ LangSmith\n\nSkip to main contentLangSmith API DocsSearchGo to AppQuick StartUser GuideTracingEvaluationProduction Monitoring & AutomationsPrompt HubProxyPricingSelf-HostingCookbookQuick StartOn this pageGetting started with LangSmithIntroduction\u200bLangSmith is a platform for building production-grade LLM applications. It allows you to closely monitor and evaluate your application, so you can ship quickly and with confidence. Use of LangChain is not necessary - LangSmith works on its own!Install LangSmith\u200bWe offer Python and Typescript SDKs for all your LangSmith needs.PythonTypeScriptpip install -U langsmithyarn add langchain langsmithCreate an API key\u200bTo create an API key head to the setting pages. Then click Create API Key.Setup your environment\u200bShellexport LANGCHAIN_TRACING_V2=trueexport LANGCHAIN_API_KEY=<your-api-key># The below examples use the OpenAI API, though it's not necessary in generalexport OPENAI_API_KEY=<your-openai-api-key>Log your first trace\u200bWe provide multiple ways to log traces\n\nLearn about the workflows LangSmith supports at each stage of the LLM application lifecycle.Pricing: Learn about the pricing model for LangSmith.Self-Hosting: Learn about self-hosting options for LangSmith.Proxy: Learn about the proxy capabilities of LangSmith.Tracing: Learn about the tracing capabilities of LangSmith.Evaluation: Learn about the evaluation capabilities of LangSmith.Prompt Hub Learn about the Prompt Hub, a prompt management tool built into LangSmith.Additional Resources\u200bLangSmith Cookbook: A collection of tutorials and end-to-end walkthroughs using LangSmith.LangChain Python: Docs for the Python LangChain library.LangChain Python API Reference: documentation to review the core APIs of LangChain.LangChain JS: Docs for the TypeScript LangChain libraryDiscord: Join us on our Discord to discuss all things LangChain!FAQ\u200bHow do I migrate projects between organizations?\u200bCurrently we do not support project migration betwen organizations. While you can manually imitate this by\n\nteam deals with sensitive data that cannot be logged. How can I ensure that only my team can access it?\u200bIf you are interested in a private deployment of LangSmith or if you need to self-host, please reach out to us at sales@langchain.dev. Self-hosting LangSmith requires an annual enterprise license that also comes with support and formalized access to the LangChain team.Was this page helpful?NextUser GuideIntroductionInstall LangSmithCreate an API keySetup your environmentLog your first traceCreate your first evaluationNext StepsAdditional ResourcesFAQHow do I migrate projects between organizations?Why aren't my runs aren't showing up in my project?My team deals with sensitive data that cannot be logged. How can I ensure that only my team can access it?CommunityDiscordTwitterGitHubDocs CodeLangSmith SDKPythonJS/TSMoreHomepageBlogLangChain Python DocsLangChain JS/TS DocsCopyright © 2024 LangChain, Inc.", name='langsmith_search', id='f286c7e7-6514-4621-ac60-e4079b37ebe2', tool_call_id='call_FNIgdO97wo51sKx3XZOGLHqT'),
 AIMessage(content="LangSmith是一个平台，可以通过提供以下几个功能显著帮助测试：\n\n1. **跟踪**：LangSmith提供强大的跟踪功能，使您能够密切监视应用程序。这个功能特别有助于跟踪应用程序的行为并识别任何潜在问题。\n\n2. **评估**：LangSmith允许您对应用程序进行全面评估。这可以帮助您评估应用程序在各种条件下的性能，并进行必要的调整以增强其功能。\n\n3. **生产监控与自动化**：使用LangSmith，您可以在应用程序处于活动状态时密切关注它。该平台提供自动监控和管理常规任务的工具，有助于确保应用程序平稳运行。\n\n4. **提示中心**：这是内置在LangSmith中的提示管理工具。在测试应用程序中的各种提示时，此功能可能非常有用。\n\n总的来说，LangSmith帮助您自信地构建生产级LLM应用程序，提供了必要的监控、评估和自动化工具。", response_metadata={'token_usage': {'completion_tokens': 200, 'prompt_tokens': 782, 'total_tokens': 982}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-4b80db7e-9a26-4043-8b6b-922f847f9c80-0')]
```
让我们查看一下 [LangSmith 追踪](https://smith.langchain.com/public/853f62d0-3421-4dba-b30a-7277ce2bdcdf/r)，看看在幕后发生了什么。
请注意，最终返回的状态还包含了工具调用和工具响应消息。
现在让我们尝试一个需要调用搜索工具的例子：
```python
response = agent_executor.invoke(
    {"messages": [HumanMessage(content="whats the weather in sf?")]}
)
response["messages"]
```
```output
[HumanMessage(content='whats the weather in sf?', id='e6b716e6-da57-41de-a227-fee281fda588'),
 AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_TGDKm0saxuGKJD5OYOXWRvLe', 'function': {'arguments': '{\n  "query": "current weather in San Francisco"\n}', 'name': 'tavily_search_results_json'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 23, 'prompt_tokens': 134, 'total_tokens': 157}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-fd7d5854-2eab-4fca-ad9e-b3de8d587614-0', tool_calls=[{'name': 'tavily_search_results_json', 'args': {'query': 'current weather in San Francisco'}, 'id': 'call_TGDKm0saxuGKJD5OYOXWRvLe'}]),
 ToolMessage(content='[{"url": "https://www.weatherapi.com/", "content": "{\'location\': {\'name\': \'San Francisco\', \'region\': \'California\', \'country\': \'United States of America\', \'lat\': 37.78, \'lon\': -122.42, \'tz_id\': \'America/Los_Angeles\', \'localtime_epoch\': 1714426800, \'localtime\': \'2024-04-29 14:40\'}, \'current\': {\'last_updated_epoch\': 1714426200, \'last_updated\': \'2024-04-29 14:30\', \'temp_c\': 17.8, \'temp_f\': 64.0, \'is_day\': 1, \'condition\': {\'text\': \'Sunny\', \'icon\': \'//cdn.weatherapi.com/weather/64x64/day/113.png\', \'code\': 1000}, \'wind_mph\': 23.0, \'wind_kph\': 37.1, \'wind_degree\': 290, \'wind_dir\': \'WNW\', \'pressure_mb\': 1019.0, \'pressure_in\': 30.09, \'precip_mm\': 0.0, \'precip_in\': 0.0, \'humidity\': 50, \'cloud\': 0, \'feelslike_c\': 17.8, \'feelslike_f\': 64.0, \'vis_km\': 16.0, \'vis_miles\': 9.0, \'uv\': 5.0, \'gust_mph\': 27.5, \'gust_kph\': 44.3}}"}, {"url": "https://www.wunderground.com/hourly/us/ca/san-francisco/94125/date/2024-4-29", "content": "Current Weather for Popular Cities . San Francisco, CA warning 59 \\u00b0 F Mostly Cloudy; Manhattan, NY 56 \\u00b0 F Fair; Schiller Park, IL (60176) warning 58 \\u00b0 F Mostly Cloudy; Boston, MA 52 \\u00b0 F Sunny ..."}]', name='tavily_search_results_json', id='aa0d8c3d-23b5-425a-ad05-3c174fc04892', tool_call_id='call_TGDKm0saxuGKJD5OYOXWRvLe'),
 AIMessage(content='The current weather in San Francisco, California is sunny with a temperature of 64.0°F (17.8°C). The wind is coming from the WNW at a speed of 23.0 mph. The humidity level is at 50%. There is no precipitation and the cloud cover is 0%. The visibility is 16.0 km. The UV index is 5.0. Please note that this information is as of 14:30 on April 29, 2024, according to [Weather API](https://www.weatherapi.com/).', response_metadata={'token_usage': {'completion_tokens': 117, 'prompt_tokens': 620, 'total_tokens': 737}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-2359b41b-cab6-40c3-b6d9-7bdf7195a601-0')]
```
我们可以查看 [LangSmith 追踪](https://smith.langchain.com/public/f520839d-cd4d-4495-8764-e32b548e235d/r) 来确保它有效地调用了搜索工具。
## 消息流
我们已经看到可以使用 `.invoke` 调用代理以获得最终响应。如果代理正在执行多个步骤，可能需要一段时间。为了显示中间进度，我们可以在发生时即时返回消息。
```python
for chunk in agent_executor.stream(
    {"messages": [HumanMessage(content="whats the weather in sf?")]}
):
    print(chunk)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_50Kb8zHmFqPYavQwF5TgcOH8', 'function': {'arguments': '{\n  "query": "current weather in San Francisco"\n}', 'name': 'tavily_search_results_json'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 23, 'prompt_tokens': 134, 'total_tokens': 157}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-042d5feb-c2cc-4c3f-b8fd-dbc22fd0bc07-0', tool_calls=[{'name': 'tavily_search_results_json', 'args': {'query': 'current weather in San Francisco'}, 'id': 'call_50Kb8zHmFqPYavQwF5TgcOH8'}])]}}
----
{'action': {'messages': [ToolMessage(content='[{"url": "https://www.weatherapi.com/", "content": "{\'location\': {\'name\': \'San Francisco\', \'region\': \'California\', \'country\': \'United States of America\', \'lat\': 37.78, \'lon\': -122.42, \'tz_id\': \'America/Los_Angeles\', \'localtime_epoch\': 1714426906, \'localtime\': \'2024-04-29 14:41\'}, \'current\': {\'last_updated_epoch\': 1714426200, \'last_updated\': \'2024-04-29 14:30\', \'temp_c\': 17.8, \'temp_f\': 64.0, \'is_day\': 1, \'condition\': {\'text\': \'Sunny\', \'icon\': \'//cdn.weatherapi.com/weather/64x64/day/113.png\', \'code\': 1000}, \'wind_mph\': 23.0, \'wind_kph\': 37.1, \'wind_degree\': 290, \'wind_dir\': \'WNW\', \'pressure_mb\': 1019.0, \'pressure_in\': 30.09, \'precip_mm\': 0.0, \'precip_in\': 0.0, \'humidity\': 50, \'cloud\': 0, \'feelslike_c\': 17.8, \'feelslike_f\': 64.0, \'vis_km\': 16.0, \'vis_miles\': 9.0, \'uv\': 5.0, \'gust_mph\': 27.5, \'gust_kph\': 44.3}}"}, {"url": "https://world-weather.info/forecast/usa/san_francisco/april-2024/", "content": "Extended weather forecast in San Francisco. Hourly Week 10 days 14 days 30 days Year. Detailed \\u26a1 San Francisco Weather Forecast for April 2024 - day/night \\ud83c\\udf21\\ufe0f temperatures, precipitations - World-Weather.info."}]', name='tavily_search_results_json', id='d88320ac-3fe1-4f73-870a-3681f15f6982', tool_call_id='call_50Kb8zHmFqPYavQwF5TgcOH8')]}}
----
{'agent': {'messages': [AIMessage(content='The current weather in San Francisco, California is sunny with a temperature of 17.8°C (64.0°F). The wind is coming from the WNW at 23.0 mph. The humidity is at 50%. [source](https://www.weatherapi.com/)', response_metadata={'token_usage': {'completion_tokens': 58, 'prompt_tokens': 602, 'total_tokens': 660}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-0cd2a507-ded5-4601-afe3-3807400e9989-0')]}}
----
```
## 流式标记
除了流式返回消息之外，流式返回标记也非常有用。
我们可以使用`.astream_events`方法来实现这一点。
:::important
这个`.astream_events`方法只适用于 Python 3.11 或更高版本。
:::
```python
async for event in agent_executor.astream_events(
    {"messages": [HumanMessage(content="whats the weather in sf?")]}, version="v1"
):
    kind = event["event"]
    if kind == "on_chain_start":
        if (
            event["name"] == "Agent"
        ):  # 创建代理时分配的`.with_config({"run_name": "Agent"})`
            print(
                f"Starting agent: {event['name']} with input: {event['data'].get('input')}"
            )
    elif kind == "on_chain_end":
        if (
            event["name"] == "Agent"
        ):  # 创建代理时分配的`.with_config({"run_name": "Agent"})`
            print()
            print("--")
            print(
                f"Done agent: {event['name']} with output: {event['data'].get('output')['output']}"
            )
    if kind == "on_chat_model_stream":
        content = event["data"]["chunk"].content
        if content:
            # 在 OpenAI 的上下文中，空内容意味着模型正在请求调用工具。
            # 因此我们只打印非空内容
            print(content, end="|")
    elif kind == "on_tool_start":
        print("--")
        print(
            f"Starting tool: {event['name']} with inputs: {event['data'].get('input')}"
        )
    elif kind == "on_tool_end":
        print(f"Done tool: {event['name']}")
        print(f"Tool output was: {event['data'].get('output')}")
        print("--")
```
```output
--
Starting tool: tavily_search_results_json with inputs: {'query': 'current weather in San Francisco'}
Done tool: tavily_search_results_json
工具输出为：[{'url': 'https://www.weatherapi.com/', 'content': "{'location': {'name': 'San Francisco', 'region': 'California', 'country': 'United States of America', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1714427052, 'localtime': '2024-04-29 14:44'}, 'current': {'last_updated_epoch': 1714426200, 'last_updated': '2024-04-29 14:30', 'temp_c': 17.8, 'temp_f': 64.0, 'is_day': 1, 'condition': {'text': 'Sunny', 'icon': '//cdn.weatherapi.com/weather/64x64/day/113.png', 'code': 1000}, 'wind_mph': 23.0, 'wind_kph': 37.1, 'wind_degree': 290, 'wind_dir': 'WNW', 'pressure_mb': 1019.0, 'pressure_in': 30.09, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 50, 'cloud': 0, 'feelslike_c': 17.8, 'feelslike_f': 64.0, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 27.5, 'gust_kph': 44.3}}"}, {'url': 'https://www.weathertab.com/en/c/e/04/united-states/california/san-francisco/', 'content': 'San Francisco Weather Forecast for Apr 2024 - Risk of Rain Graph. Rain Risk Graph: Monthly Overview. Bar heights indicate rain risk percentages. Yellow bars mark low-risk days, while black and grey bars signal higher risks. Grey-yellow bars act as buffers, advising to keep at least one day clear from the riskier grey and black days, guiding ...'}]
--
当前|旧金山|,|加利福尼亚州|,|美国|的|天气|晴朗|，|气温|为|17.8|°C|（|64.0|°F|）。|风速|为|37.1|千米/小时|（|23.0|英里/小时|），|来自|西北偏西|。|湿度|为|50|%|。| [来源](https://www.weatherapi.com/)
```
## 添加内存
如前所述，此代理是无状态的。这意味着它不会记住先前的交互。要给它添加记忆，我们需要传入一个检查点。在传入检查点时，还必须在调用代理时传入一个`thread_id`（以便它知道从哪个线程/对话中恢复）。
```python
from langgraph.checkpoint.sqlite import SqliteSaver
memory = SqliteSaver.from_conn_string(":memory:")
```
```python
agent_executor = chat_agent_executor.create_tool_calling_executor(
    model, tools, checkpointer=memory
)
config = {"configurable": {"thread_id": "abc123"}}
```
```python
for chunk in agent_executor.stream(
    {"messages": [HumanMessage(content="hi im bob!")]}, config
):
    print(chunk)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 11, 'prompt_tokens': 131, 'total_tokens': 142}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-607733e3-4b8d-4137-ae66-8a4b8ccc8d40-0')]}}
----
```
```python
for chunk in agent_executor.stream(
    {"messages": [HumanMessage(content="whats my name?")]}, config
):
    print(chunk)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='Your name is Bob. How can I assist you further?', response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 154, 'total_tokens': 167}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-e1181ba6-732d-4564-b479-9f1ab6bf01f6-0')]}}
----
```
示例[LangSmith跟踪](https://smith.langchain.com/public/fa73960b-0f7d-4910-b73d-757a12f33b2b/r)
## 结论
就是这样！在这个快速入门中，我们介绍了如何创建一个简单的代理。
然后，我们展示了如何返回响应流 - 不仅包括中间步骤，还包括标记！
我们还添加了内存，这样您就可以与其进行对话。
代理是一个复杂的主题，有很多东西要学习！
有关代理的更多信息，请查看[LangGraph](/docs/concepts/#langgraph)文档。其中包含一套概念、教程和操作指南。