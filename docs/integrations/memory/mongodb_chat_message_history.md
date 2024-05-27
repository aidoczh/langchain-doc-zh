# MongoDB

`MongoDB` 是一款开源的跨平台文档导向型数据库程序。作为一款NoSQL数据库程序，`MongoDB` 使用类似于 `JSON` 的文档，并可选择使用模式。

`MongoDB` 由 MongoDB Inc. 开发，并在 Server Side Public License (SSPL) 下获得许可。- [维基百科](https://en.wikipedia.org/wiki/MongoDB)

本文介绍了如何使用 `MongoDBChatMessageHistory` 类将聊天消息历史存储在 MongoDB 数据库中。

## 设置

集成在 `langchain-mongodb` 包中，因此我们需要安装该包。

```bash
pip install -U --quiet langchain-mongodb
```

为了获得最佳的可观察性，设置 [LangSmith](https://smith.langchain.com/) 也会很有帮助（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

要使用存储功能，您只需要提供两个东西：

1. 会话 ID - 会话的唯一标识符，例如用户名、电子邮件、聊天 ID 等。

2. 连接字符串 - 一个指定数据库连接的字符串。它将传递给 MongoDB create_engine 函数。

如果您想自定义聊天历史记录的存储位置，还可以传递以下参数：

1. *database_name* - 要使用的数据库的名称

2. *collection_name* - 在该数据库中要使用的集合

```python
from langchain_mongodb.chat_message_histories import MongoDBChatMessageHistory
chat_message_history = MongoDBChatMessageHistory(
    session_id="test_session",
    connection_string="mongodb://mongo_user:password123@mongo:27017",
    database_name="my_db",
    collection_name="chat_histories",
)
chat_message_history.add_user_message("Hello")
chat_message_history.add_ai_message("Hi")
```

```python
chat_message_history.messages
```

```output
[HumanMessage(content='Hello'), AIMessage(content='Hi')]
```

## 链式调用

我们可以将这个消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 轻松结合起来。

为此，我们将使用 OpenAI，因此我们需要安装它。您还需要将 OPENAI_API_KEY 环境变量设置为您的 OpenAI 密钥。

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
```

```python
import os
assert os.environ[
    "OPENAI_API_KEY"
], "Set the OPENAI_API_KEY environment variable with your OpenAI API key."
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: MongoDBChatMessageHistory(
        session_id=session_id,
        connection_string="mongodb://mongo_user:password123@mongo:27017",
        database_name="my_db",
        collection_name="chat_histories",
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这是我们配置会话 ID 的地方
config = {"configurable": {"session_id": "<SESSION_ID>"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```output
AIMessage(content='Hi Bob! How can I assist you today?')
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content='Your name is Bob. Is there anything else I can help you with, Bob?')
```