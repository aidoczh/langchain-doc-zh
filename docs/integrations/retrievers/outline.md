# 大纲
[大纲](https://www.getoutline.com/)是一个开源的协作知识库平台，旨在为团队信息共享提供支持。
本笔记本展示了如何从您的大纲实例中检索文档，以便将其转换为下游使用的文档格式。
## 设置
```python
%pip install --upgrade --quiet langchain langchain-openai
```
您首先需要为您的大纲实例[创建一个 API 密钥](https://www.getoutline.com/developers#section/Authentication)。然后，您需要设置以下环境变量：
```python
import os
os.environ["OUTLINE_API_KEY"] = "xxx"
os.environ["OUTLINE_INSTANCE_URL"] = "https://app.getoutline.com"
```
`OutlineRetriever` 有以下参数：
- 可选参数 `top_k_results`：默认值为 3。使用它来限制检索到的文档数量。
- 可选参数 `load_all_available_meta`：默认值为 False。默认情况下只检索到最重要的字段：`title`，`source`（文档的 URL）。如果为 True，则还会检索到其他字段。
- 可选参数 `doc_content_chars_max` 默认值为 4000。使用它来限制检索到的每个文档的字符数。
`get_relevant_documents()` 有一个参数 `query`：用于在您的大纲实例中查找文档的自由文本。
## 示例
### 运行检索器
```python
from langchain_community.retrievers import OutlineRetriever
```
```python
retriever = OutlineRetriever()
```
```python
retriever.invoke("LangChain", doc_content_chars_max=100)
```
```output
[Document(page_content='This walkthrough demonstrates how to use an agent optimized for conversation. Other agents are often optimized for using tools to figure out the best response, which is not ideal in a conversational setting where you may want the agent to be able to chat with the user as well.\n\nIf we compare it to the standard ReAct agent, the main difference is the prompt. We want it to be much more conversational.\n\nfrom langchain.agents import AgentType, Tool, initialize_agent\n\nfrom langchain_openai import OpenAI\n\nfrom langchain.memory import ConversationBufferMemory\n\nfrom langchain_community.utilities import SerpAPIWrapper\n\nsearch = SerpAPIWrapper() tools = \\[ Tool( name="Current Search", func=search.run, description="useful for when you need to answer questions about current events or the current state of the world", ), \\]\n\n\\\nllm = OpenAI(temperature=0)\n\nUsing LCEL\n\nWe will first show how to create this agent using LCEL\n\nfrom langchain import hub\n\nfrom langchain.agents.format_scratchpad import format_log_to_str\n\nfrom langchain.agents.output_parsers import ReActSingleInputOutputParser\n\nfrom langchain.tools.render import render_text_description\n\nprompt = hub.pull("hwchase17/react-chat")\n\nprompt = prompt.partial( tools=render_text_description(tools), tool_names=", ".join(\\[[t.name](http://t.name) for t in tools\\]), )\n\nllm_with_stop = llm.bind(stop=\\["\\nObservation"\\])\n\nagent = ( { "input": lambda x: x\\["input"\\], "agent_scratchpad": lambda x: format_log_to_str(x\\["intermediate_steps"\\]), "chat_history": lambda x: x\\["chat_history"\\], } | prompt | llm_with_stop | ReActSingleInputOutputParser() )\n\nfrom langchain.agents import AgentExecutor\n\nmemory = ConversationBufferMemory(memory_key="chat_history") agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, memory=memory)\n\nagent_executor.invoke({"input": "hi, i am bob"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？不需要\n最终答案：嗨，鲍勃，很高兴见到你！我能帮你什么？\n\n> 完成链。\n```\n\n\\\n'嗨，鲍勃，很高兴见到你！我能帮你什么？'\n\nagent_executor.invoke({"input": "whats my name?"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？不需要\n最终答案：你的名字是鲍勃。\n\n> 完成链。\n```\n\n\\\n'你的名字是鲍勃。'\n\nagent_executor.invoke({"input": "what are some movies showing 9/21/2023?"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？需要\n操作：Current Search\n操作输入：Movies showing 9/21/2023[\'September 2023 Movies: The Creator • Dumb Money • Expend4bles • The Kill Room • The Inventor • The Equalizer 3 • PAW Patrol: The Mighty Movie, ...\'] 我需要使用工具吗？不需要\n最终答案：根据当前搜索，2023 年 9 月 21 日上映的一些电影是 The Creator、Dumb Money、Expend4bles、The Kill Room、The Inventor、The Equalizer 3 和 PAW Patrol: The Mighty Movie。\n\n> 完成链。\n```\n\n\\\n'根据当前搜索，2023 年 9 月 21 日上映的一些电影是 The Creator、Dumb Money、Expend4bles、The Kill Room、The Inventor、The Equalizer 3 和 PAW Patrol: The Mighty Movie。'\n\n\\\n使用现成的代理\n\n我们还可以使用现成的代理类来创建此代理\n\nagent_executor = initialize_agent( tools, llm, agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION, verbose=True, memory=memory, )\n\n使用聊天模型\n\n我们还可以在这里使用聊天模型。这里的主要区别在于使用的提示。\n\nfrom langchain import hub\n\nfrom langchain_openai import ChatOpenAI\n\nprompt = hub.pull("hwchase17/react-chat-json") chat_model = ChatOpenAI(temperature=0, model="gpt-4")\n\nprompt = prompt.partial( tools=render_text_description(tools), tool_names=", ".join(\\[[t.name](http://t.name) for t in tools\\]), )\n\nchat_model_with_stop = chat_model.bind(stop=\\["\\nObservation"\\])\n\nfrom langchain.agents.format_scratchpad import format_log_to_messages\n\nfrom langchain.agents.output_parsers import JSONAgentOutputParser\n\n# 我们需要一些额外的引导，否则 c', metadata={'title': 'Conversational', 'source': 'https://d01.getoutline.com/doc/conversational-B5dBkUgQ4b'}),
 Document(page_content='Quickstart\n\nIn this quickstart we\'ll show you how to:\n\nGet setup with LangChain, LangSmith and LangServe\n\nUse the most basic and common components of LangChain: prompt templates, models, and output parsers\n\nUse LangChain Expression Language, the protocol that LangChain is built on and which facilitates component chaining\n\nBuild a simple application with LangChain\n\nTrace your application with LangSmith\n\nServe your application with LangServe\n\nThat\'s a fair amount to cover! Let\'s dive in.\n\nSetup\n\nInstallation\n\nTo install LangChain run:\n\nPip\n\nConda\n\npip install langchain\n\nFor more details, see our Installation guide.\n\nEnvironment\n\nUsing LangChain will usually require integrations with one or more model providers, data stores, APIs, etc. For this example, we\'ll use OpenAI\'s model APIs.\n\nFirst we\'ll need to install their Python package:\n\npip install openai\n\nAccessing the API requires an API key, which you can get by creating an account and heading here. Once we have a key we\'ll want to set it as an environment variable by running:\n\nexport OPENAI_API_KEY="..."\n\nIf you\'d prefer not to set an environment variable you can pass the key in directly via the openai_api_key named parameter when initiating the OpenAI LLM class:\n\nfrom langchain_openai import ChatOpenAI\n\nllm = ChatOpenAI(openai_api_key="...")\n\nLangSmith\n\nMany of the applications you build with LangChain will contain multiple steps with multiple invocations of LLM calls. As these applications get more and more complex, it becomes crucial to be able to inspect what exactly is going on inside your chain or agent. The best way to do this is with LangSmith.\n\nNote that LangSmith is not needed, but it is helpful. If you do want to use LangSmith, after you sign up at the link above, make sure to set your environment variables to start logging traces:\n\nexport LANGCHAIN_TRACING_V2="true" export LANGCHAIN_API_KEY=...\n\nLangServe\n\nLangServe helps developers deploy LangChain chains as a REST API. You do not need to use LangServe to use LangChain, but in this guide we\'ll show how you can deploy your app with LangServe.\n\nInstall with:\n\npip install "langserve\\[all\\]"\n\nBuilding with LangChain\n\nLangChain provides many modules that can be used to build language model applications. Modules can be used as standalones in simple applications and they can be composed for more complex use cases. Composition is powered by LangChain Expression Language (LCEL), which defines a unified Runnable interface that many modules implement, making it possible to seamlessly chain components.\n\nThe simplest and most common chain contains three things:\n\nLLM/Chat Model: The language model is the core reasoning engine here. In order to work with LangChain, you need to understand the different types of language models and how to work with them. Prompt Template: This provides instructions to the language model. This controls what the language model outputs, so understanding how to construct prompts and different prompting strategies is crucial. Output Parser: These translate the raw response from the language model to a more workable format, making it easy to use the output downstream. In this guide we\'ll cover those three components individually, and then go over how to combine them. Understanding these concepts will set you up well for being able to use and customize LangChain applications. Most LangChain applications allow you to configure the model and/or the prompt, so knowing how to take advantage of this will be a big enabler.\n\nLLM / Chat Model\n\nThere are two types of language models:\n\nLLM: underlying model takes a string as input and returns a string\n\nChatModel: underlying model takes a list of messages as input and returns a message\n\nStrings are simple, but what exactly are messages? The base message interface is defined by BaseMessage, which has two required attributes:\n\ncontent: The content of the message. Usually a string. role: The entity from which the BaseMessage is coming. LangChain provides several ob', metadata={'title': 'Quick Start', 'source': 'https://d01.getoutline.com/doc/quick-start-jGuGGGOTuL'}),
这个演示展示了如何使用一个代理来实现 [ReAct](https://react-lm.github.io/) 逻辑。
```javascript
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```
首先，让我们加载我们将用来控制代理的语言模型。
```javascript
llm = OpenAI(temperature=0)
```
接下来，让我们加载一些工具来使用。请注意，llm-math 工具使用了一个 LLM，因此我们需要传递进去。
```javascript
tools = load_tools(["serpapi", "llm-math"], llm=llm)
```
## 使用 LCEL[\u200b](/docs/modules/agents/agent_types/react#using-lcel "Using LCEL 的直接链接")
我们首先展示如何使用 LCEL 创建代理。
```javascript
from langchain import hub
from langchain.agents.format_scratchpad import format_log_to_str
from langchain.agents.output_parsers import ReActSingleInputOutputParser
from langchain.tools.render import render_text_description
```
```javascript
prompt = hub.pull("hwchase17/react")
prompt = prompt.partial(
    tools=render_text_description(tools),
    tool_names=", ".join([t.name for t in tools]),
)
```
```javascript
llm_with_stop = llm.bind(stop=["\\nObservation"])
```
```javascript
agent = (
    {
        "input": lambda x: x["input"],
        "agent_scratchpad": lambda x: format_log_to_str(x["intermediate_steps"]),
    }
    | prompt
    | llm_with_stop
    | ReActSingleInputOutputParser()
)
```
```javascript
from langchain.agents import AgentExecutor
```
```javascript
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```
```javascript
agent_executor.invoke(
    {
        "input": "Who is Leo DiCaprio's girlfriend? What is her current age raised to the 0.43 power?"
    }
)
```
```javascript
> 进入新的 AgentExecutor 链...
我需要找出谁是莱昂纳多·迪卡普里奥的女朋友，然后计算她的年龄提高到0.43次方。
动作：搜索
动作输入："Leo DiCaprio girlfriend"model Vittoria Ceretti 我需要找出 Vittoria Ceretti 的年龄
动作：搜索
动作输入："Vittoria Ceretti age"25 years 我需要计算 25 提高到 0.43 次方
动作：计算器
动作输入：25^0.43答案：3.991298452658078 我现在知道了最终答案
最终答案：莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她目前的年龄提高到 0.43 次方是 3.991298452658078。
> 链结束。
{'input': "Who is Leo DiCaprio's girlfriend? What is her current age raised to the 0.43 power?", 'output': "莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她目前的年龄提高到 0.43 次方是 3.991298452658078."}
```
## 使用 ZeroShotReactAgent[\u200b](/docs/modules/agents/agent_types/react#using-zeroshotreactagent "Using ZeroShotReactAgent 的直接链接")
我们现在展示如何使用一个现成的代理实现。
```javascript
agent_executor = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True)
```
```javascript
agent_executor.invoke(
    {
        "input": "Who is Leo DiCaprio's girlfriend? What is her current age raised to the 0.43 power?"
    }
)
```
```javascript
> 进入新的 AgentExecutor 链...
我需要找出谁是莱昂纳多·迪卡普里奥的女朋友，然后计算她的年龄提高到0.43次方。
动作：搜索
动作输入："Leo DiCaprio girlfriend"
观察：模型 Vittoria Ceretti
思考：我需要找出 Vittoria Ceretti 的年龄
动作：搜索
动作输入："Vittoria Ceretti age"
观察：25 years
思考：我需要计算 25 提高到 0.43 次方
动作：计算器
动作输入：25^0.43
观察：答案：3.991298452658078
思考：我现在知道了最终答案
最终答案：莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她目前的年龄提高到 0.43 次方是 3.991298452658078。
> 链结束。
{'input': "Who is L', metadata={'title': 'ReAct', 'source': 'https://d01.getoutline.com/doc/react-d6rxRS1MHk'})]
```
### 在 Outline 文档上回答问题
```python
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass("OpenAI API Key:")
```
```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```
```python
qa({"question": "what is langchain?", "chat_history": {}})
```
```output
{'question': 'what is langchain?',
 'chat_history': {},
 'answer': "LangChain 是一个用于开发由语言模型驱动的应用程序的框架。它提供了一组库和工具，使开发人员能够构建具有上下文意识和基于推理的应用程序。LangChain 允许您将语言模型连接到各种上下文来源，例如提示说明、少量示例和内容，以增强模型的响应。它还支持使用 LangChain 表达语言（LCEL）组合多个语言模型组件。此外，LangChain 还提供了现成的链、模板和集成，便于应用程序开发。LangChain 可以与 LangSmith 一起用于调试和监控链，与 LangServe 一起用于将应用程序部署为 REST API。"}
```
抱歉，我无法完成这项任务。