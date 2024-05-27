# Xata

[Xata](https://xata.io) 是一个基于 `PostgreSQL` 和 `Elasticsearch` 的无服务器数据平台。它提供了一个 Python SDK 用于与数据库交互，并提供了一个用户界面用于管理数据。通过 `XataChatMessageHistory` 类，您可以使用 Xata 数据库来长期保存聊天会话。

本笔记涵盖了以下内容：

- 一个简单的示例，展示了 `XataChatMessageHistory` 的功能。

- 一个更复杂的示例，使用一个基于知识库或文档的 REACT 代理来回答问题（存储在 Xata 中作为向量存储），并且具有其过去消息的长期可搜索历史记录（存储在 Xata 中作为内存存储）。

## 设置

### 创建数据库

在[Xata 用户界面](https://app.xata.io)中创建一个新数据库。您可以随意命名，本示例中我们将使用 `langchain`。Langchain 集成可以自动创建用于存储内存的表，这是我们在本示例中将使用的功能。如果您想要预先创建表，请确保它具有正确的模式，并在创建类时将 `create_table` 设置为 `False`。预先创建表可以节省每次会话初始化时与数据库的一次往返。

让我们首先安装我们的依赖项：

```python
%pip install --upgrade --quiet  xata langchain-openai langchain
```

接下来，我们需要获取 Xata 的环境变量。您可以通过访问您的[账户设置](https://app.xata.io/settings)来创建一个新的 API 密钥。要找到数据库 URL，请转到您创建的数据库的设置页面。数据库 URL 应该看起来像这样：`https://demo-uni3q8.eu-west-1.xata.sh/db/langchain`。

```python
import getpass
api_key = getpass.getpass("Xata API key: ")
db_url = input("Xata database URL (copy it from your DB settings):")
```

## 创建一个简单的内存存储

为了独立测试内存存储功能，让我们使用以下代码片段：

```python
from langchain_community.chat_message_histories import XataChatMessageHistory
history = XataChatMessageHistory(
    session_id="session-1", api_key=api_key, db_url=db_url, table_name="memory"
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

上述代码创建了一个 ID 为 `session-1` 的会话，并将两条消息存储在其中。运行上述代码后，如果您访问 Xata 用户界面，您应该会看到一个名为 `memory` 的表以及其中添加的两条消息。

您可以使用以下代码检索特定会话的消息历史记录：

```python
history.messages
```

## 与内存进行对话的问答链

现在让我们看一个更复杂的示例，其中我们结合了 OpenAI、Xata 向量存储集成和 Xata 内存存储集成，以创建一个在您的数据上进行问答的聊天机器人，具有后续问题和历史记录。

我们将需要访问 OpenAI API，因此让我们配置 API 密钥：

```python
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

要存储聊天机器人将搜索答案的文档，使用 Xata 用户界面向您的 `langchain` 数据库添加一个名为 `docs` 的表，并添加以下列：

- `content` 类型为 "Text"。这用于存储 `Document.pageContent` 的值。

- `embedding` 类型为 "Vector"。使用您计划使用的模型的维度。在本笔记中，我们使用 OpenAI 嵌入，它有 1536 个维度。

让我们创建向量存储并向其添加一些示例文档：

```python
from langchain_community.vectorstores.xata import XataVectorStore
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
texts = [
    "Xata is a Serverless Data platform based on PostgreSQL",
    "Xata offers a built-in vector type that can be used to store and query vectors",
    "Xata includes similarity search",
]
vector_store = XataVectorStore.from_texts(
    texts, embeddings, api_key=api_key, db_url=db_url, table_name="docs"
)
```

运行上述命令后，如果您转到 Xata 用户界面，您应该会看到加载的文档及其嵌入在 `docs` 表中。

现在让我们创建一个 ConversationBufferMemory 来存储用户和 AI 的聊天消息：

```python
from uuid import uuid4
from langchain.memory import ConversationBufferMemory
chat_memory = XataChatMessageHistory(
    session_id=str(uuid4()),  # 每个用户会话需要是唯一的
    api_key=api_key,
    db_url=db_url,
    table_name="memory",
)
memory = ConversationBufferMemory(
    memory_key="chat_history", chat_memory=chat_memory, return_messages=True
)
```

现在是时候创建一个 Agent 来同时使用向量存储和聊天内存了：

```python
from langchain.agents import AgentType, initialize_agent
from langchain.agents.agent_toolkits import create_retriever_tool
from langchain_openai import ChatOpenAI
tool = create_retriever_tool(
    vector_store.as_retriever(),
    "search_docs",
    "Searches and returns documents from the Xata manual. Useful when you need to answer questions about Xata.",
)
tools = [tool]
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.CHAT_CONVERSATIONAL_REACT_DESCRIPTION,
    verbose=True,
    memory=memory,
)
```

为了测试，让我们告诉这个代理我们的名字：

```python
agent.run(input="My name is bob")
```

现在，让我们问代理一些关于 Xata 的问题：

```python
agent.run(input="What is xata?")
```

注意它的回答是基于文档存储中存储的数据。现在，让我们问一个后续问题：

```python
agent.run(input="Does it support similarity search?")
```

现在让我们测试它的记忆力：

```python
agent.run(input="Did I tell you my name? What is it?")
```

```