---
sidebar_position: 1
---
# 构建一个聊天机器人
## 概述
我们将介绍如何设计和实现一个由LLM驱动的聊天机器人的示例。
这个聊天机器人将能够进行对话并记住先前的交互。
请注意，我们构建的这个聊天机器人将仅使用语言模型进行对话。
您可能还在寻找其他相关概念：
- [对话RAG](/docs/tutorials/qa_chat_history)：在外部数据源上启用聊天机器人体验
- [代理](/docs/tutorials/agents)：构建可以执行操作的聊天机器人
本教程将涵盖基础知识，这对于这两个更高级的主题将是有帮助的，但如果您选择，也可以直接跳转到那里。
## 概念
以下是我们将要使用的一些高级组件：
- [`聊天模型`](/docs/concepts/#chat-models)。聊天机器人接口是基于消息而不是原始文本构建的，因此最适合于聊天模型而不是文本LLM。
- [`提示模板`](/docs/concepts/#prompt-templates)，简化了组装提示的过程，这些提示结合了默认消息、用户输入、聊天历史和（可选）额外检索到的上下文。
- [`聊天历史`](/docs/concepts/#chat-history)，允许聊天机器人“记住”过去的交互，并在回答后续问题时考虑它们。
- 使用[LangSmith](/docs/concepts/#langsmith)调试和跟踪您的应用程序
我们将介绍如何将上述组件组合在一起，创建一个强大的对话聊天机器人。
## 设置
### Jupyter Notebook
本指南（以及文档中的大多数其他指南）使用[Jupyter笔记本](https://jupyter.org/)，并假设读者也在使用。Jupyter笔记本非常适合学习如何使用LLM系统，因为通常会出现问题（意外输出、API宕机等），在交互式环境中阅读指南是更好地理解它们的好方法。
这个和其他教程可能最方便在Jupyter笔记本中运行。请参阅[此处](https://jupyter.org/install)以获取安装说明。
### 安装
要安装LangChain，请运行：

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
使用LangChain构建的许多应用程序将包含多个步骤，其中会多次调用LLM。
随着这些应用程序变得越来越复杂，能够检查链或代理内部发生了什么变得至关重要。
这样做的最佳方式是使用[LangSmith](https://smith.langchain.com)。
在上面的链接上注册后，请确保设置您的环境变量以开始记录跟踪：
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
## 快速开始
首先，让我们学习如何单独使用语言模型。LangChain支持许多可以互换使用的不同语言模型-选择您想要使用的模型！
```python
<ChatModelTabs openaiParams={`model="gpt-3.5-turbo"`} />
```
让我们首先直接使用模型。`ChatModel`是LangChain的“可运行”实例，这意味着它们公开了一个与它们交互的标准接口。要仅仅调用模型，我们可以将消息列表传递给`.invoke`方法。
```python
from langchain_core.messages import HumanMessage
model.invoke([HumanMessage(content="Hi! I'm Bob")])
```
```output
AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 12, 'total_tokens': 22}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-be38de4a-ccef-4a48-bf82-4292510a8cbf-0')
```
模型本身没有任何状态概念。例如，如果您问一个后续问题：
```python
model.invoke([HumanMessage(content="What's my name?")])
```
```output
AIMessage(content="I'm sorry, as an AI assistant, I do not have the capability to know your name unless you provide it to me.", response_metadata={'token_usage': {'completion_tokens': 26, 'prompt_tokens': 12, 'total_tokens': 38}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_caf95bb1ae', 'finish_reason': 'stop', 'logprobs': None}, id='run-8d8a9d8b-dddb-48f1-b0ed-ce80ce5397d8-0')
```
让我们来看一个示例 [LangSmith trace](https://smith.langchain.com/public/5c21cb92-2814-4119-bae9-d02b8db577ac/r)。
我们可以看到，它没有将先前的对话转换为上下文，并且无法回答问题。
这会导致糟糕的聊天机器人体验！
为了解决这个问题，我们需要将整个对话历史传递给模型。让我们看看这样做会发生什么：
```python
from langchain_core.messages import AIMessage
model.invoke(
    [
        HumanMessage(content="Hi! I'm Bob"),
        AIMessage(content="Hello Bob! How can I assist you today?"),
        HumanMessage(content="What's my name?"),
    ]
)
```
```output
AIMessage(content='Your name is Bob.', response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 35, 'total_tokens': 40}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-5692718a-5d29-4f84-bad1-a9819a6118f1-0')
```
现在我们可以看到我们得到了一个良好的回复！
这是支持聊天机器人进行对话互动的基本思想。
那么我们该如何最好地实现这一点呢？
## 消息历史
我们可以使用消息历史类来包装我们的模型并使其具有状态。
这将跟踪模型的输入和输出，并将它们存储在某个数据存储中。
未来的交互将加载这些消息并将它们作为输入的一部分传递给链。
让我们看看如何使用它！
首先，让我们确保安装了 `langchain-community`，因为我们将在其中使用集成来存储消息历史。
```python
# ! pip install langchain_community
```
之后，我们可以导入相关类并设置包装模型并添加此消息历史的链。这里的一个关键部分是我们作为 `get_session_history` 传递的函数。这个函数预期接受一个 `session_id` 并返回一个消息历史对象。这个 `session_id` 用于区分不同的对话，并应在调用新链时作为配置的一部分传递（我们将展示如何做到这一点）。
```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
store = {}
def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]
with_message_history = RunnableWithMessageHistory(model, get_session_history)
```
现在我们需要创建一个 `config`，每次传递给可运行对象。这个配置包含的信息不是直接的输入，但仍然很有用。在这种情况下，我们想要包括一个 `session_id`。它应该看起来像这样：
```python
config = {"configurable": {"session_id": "abc2"}}
```
```python
response = with_message_history.invoke(
    [HumanMessage(content="Hi! I'm Bob")],
    config=config,
)
response.content
```
```output
'Hello Bob! How can I assist you today?'
```
```python
response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)
response.content
```
```output
'Your name is Bob.'
```
太棒了！我们的聊天机器人现在记住了关于我们的事情。如果我们更改配置以引用不同的 `session_id`，我们可以看到它会重新开始对话。
```python
config = {"configurable": {"session_id": "abc3"}}
response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)
response.content
```
```output
"我很抱歉，除非您告诉我，否则我无法知道您的名字。"
```
然而，我们总是可以回到原始对话（因为我们在数据库中持久化了它）。
```python
config = {"configurable": {"session_id": "abc2"}}
response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)
response.content
```
```output
'Your name is Bob.'
```
这就是我们如何支持聊天机器人与许多用户进行对话！
现在，我们所做的一切只是在模型周围添加了一个简单的持久化层。我们可以开始通过添加提示模板使其更复杂和个性化。
## 提示模板
提示模板有助于将原始用户信息转换为 LLM 可以处理的格式。在这种情况下，原始用户输入只是一条消息，我们将其传递给 LLM。现在让我们将其变得更加复杂。首先，让我们添加一个带有一些自定义说明的系统消息（但仍然以消息作为输入）。接下来，我们将添加除了消息之外的更多输入。
首先，让我们添加一个系统消息。为此，我们将创建一个 ChatPromptTemplate。我们将利用 `MessagesPlaceholder` 来传递所有消息。
```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个乐于助人的助手。尽力回答所有问题。",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
chain = prompt | model
```
请注意，这里稍微改变了输入类型 - 不再传入消息列表，而是传入一个包含 `messages` 键的字典，其中包含一个消息列表。
```python
response = chain.invoke({"messages": [HumanMessage(content="嗨！我是鲍勃")]})
response.content
```
```output
'你好，鲍勃！今天我能为你做些什么？'
```
现在，我们可以像之前一样将其包装在相同的消息历史对象中。
```python
with_message_history = RunnableWithMessageHistory(chain, get_session_history)
```
```python
config = {"configurable": {"session_id": "abc5"}}
```
```python
response = with_message_history.invoke(
    [HumanMessage(content="嗨！我是吉姆")],
    config=config,
)
response.content
```
```output
'你好，吉姆！今天我能为你做些什么？'
```
```python
response = with_message_history.invoke(
    [HumanMessage(content="我的名字是什么？")],
    config=config,
)
response.content
```
```output
'你的名字是吉姆。吉姆，我还能帮你什么吗？'
```
太棒了！现在让我们让我们的提示变得更加复杂一些。假设提示模板现在如下所示：
```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个乐于助人的助手。尽力回答所有问题，使用 {language}。",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
chain = prompt | model
```
请注意，我们在提示中添加了一个新的 `language` 输入。现在，我们可以调用链并传入我们选择的语言。
```python
response = chain.invoke(
    {"messages": [HumanMessage(content="嗨！我是鲍勃")], "language": "西班牙语"}
)
response.content
```
```output
'¡Hola，鲍勃！¿En qué puedo ayudarte hoy?'
```
现在，让我们将这个更复杂的链包装在一个消息历史类中。这次，由于输入中有多个键，我们需要指定正确的键来保存聊天历史记录。
```python
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="messages",
)
```
```python
config = {"configurable": {"session_id": "abc11"}}
```
```python
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="嗨！我是托德")], "language": "西班牙语"},
    config=config,
)
response.content
```
```output
'¡Hola，托德！¿En qué puedo ayudarte hoy?'
```
```python
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="我的名字是什么？")], "language": "西班牙语"},
    config=config,
)
response.content
```
```output
'Tu nombre es托德。¿Hay algo más en lo que pueda ayudarte?'
```
为了帮助你理解内部发生的事情，请查看[这个 LangSmith 跟踪](https://smith.langchain.com/public/f48fabb6-6502-43ec-8242-afc352b769ed/r)。
## 管理对话历史
构建聊天机器人时，一个重要的概念是如何管理对话历史。如果不加以管理，消息列表将无限增长，并有可能超出 LLM 的上下文窗口。因此，重要的是在提示模板之前但在从消息历史加载之前添加一个限制传入消息大小的步骤。
我们可以通过在提示之前添加一个简单的步骤来修改适当的 `messages` 键，然后将该新链包装在消息历史类中。首先，让我们定义一个函数来修改传入的消息。让我们使其选择最近的 `k` 条消息。然后，我们可以通过在开头添加该函数来创建一个新的链。
```python
from langchain_core.runnables import RunnablePassthrough
def filter_messages(messages, k=10):
    return messages[-k:]
chain = (
    RunnablePassthrough.assign(messages=lambda x: filter_messages(x["messages"]))
    | prompt
    | model
)
```
现在让我们试试！如果我们创建一个超过 10 条消息的消息列表，我们可以看到它不再记得早期消息中的信息。
```python
messages = [
    HumanMessage(content="嗨！我是鲍勃"),
    AIMessage(content="嗨！"),
    HumanMessage(content="我喜欢香草冰淇淋"),
    AIMessage(content="不错"),
    HumanMessage(content="2 + 2 等于多少"),
    AIMessage(content="4"),
    HumanMessage(content="谢谢"),
    AIMessage(content="没问题！"),
    HumanMessage(content="玩得开心吗？"),
    AIMessage(content="是的！"),
]
```
```python
response = chain.invoke(
    {
        "messages": messages + [HumanMessage(content="我的名字是什么？")],
        "language": "中文",
    }
)
response.content
```
```output
"很抱歉，我无法获取您的名字。我还能帮您其他什么吗？"
```
但是，如果我们询问的信息在最近的十条消息中，它仍然会记住它
```python
response = chain.invoke(
    {
        "messages": messages + [HumanMessage(content="我最喜欢的冰淇淋是什么？")],
        "language": "中文",
    }
)
response.content
```
```output
'您提到您喜欢香草口味的冰淇淋。'
```
现在让我们将其包装在消息历史中
```python
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="messages",
)
config = {"configurable": {"session_id": "abc20"}}
```
```python
response = with_message_history.invoke(
    {
        "messages": messages + [HumanMessage(content="我的名字是什么？")],
        "language": "中文",
    },
    config=config,
)
response.content
```
```output
"很抱歉，我不知道您的名字。"
```
现在聊天历史中有两条新消息。这意味着我们以前对话历史中可访问的更多信息现在不再可用！
```python
response = with_message_history.invoke(
    {
        "messages": [HumanMessage(content="我最喜欢的冰淇淋是什么？")],
        "language": "中文",
    },
    config=config,
)
response.content
```
```output
"很抱歉，我不知道您最喜欢的冰淇淋口味。"
```
如果您查看 LangSmith，您可以在[LangSmith跟踪](https://smith.langchain.com/public/fa6b00da-bcd8-4c1c-a799-6b32a3d62964/r)中看到底层发生的情况。
## 流式传输
现在我们有了一个聊天机器人函数。然而，聊天机器人应用程序的一个*非常*重要的用户体验考虑因素是流式传输。LLM有时需要一段时间才能回复，为了提高用户体验，大多数应用程序会将每个生成的令牌流式传输回来。这样用户就可以看到进展。
实际上，这非常容易做到！
所有的链都暴露了一个`.stream`方法，使用消息历史的链也不例外。我们可以简单地使用该方法来获得流式响应。
```python
config = {"configurable": {"session_id": "abc15"}}
for r in with_message_history.stream(
    {
        "messages": [HumanMessage(content="你好！我是Todd。给我讲个笑话")],
        "language": "中文",
    },
    config=config,
):
    print(r.content, end="|")
```
```output
|当然|，| Todd|！| 这里|有个|笑话|给你|：|为什么|科学家|不|相信|原子|？|因为|它们|组成|了|一切|！|
```
## 下一步
现在您已经了解了如何在LangChain中创建聊天机器人的基础知识，您可能会对一些更高级的教程感兴趣：
- [对话RAG](/docs/tutorials/qa_chat_history)：在外部数据源上启用聊天机器人体验
- [代理](/docs/tutorials/agents)：构建可以执行操作的聊天机器人
如果您想深入了解具体内容，一些值得查看的内容包括：
- [流式传输](/docs/how_to/streaming)：流式传输对于聊天应用程序非常重要
- [如何添加消息历史记录](/docs/how_to/message_history)：深入了解与消息历史记录相关的所有内容