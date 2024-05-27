# SQLite

[SQLite](https://en.wikipedia.org/wiki/SQLite) 是用 C 语言编写的数据库引擎。它不是一个独立的应用程序，而是一个软件开发人员嵌入其应用程序中的库。因此，它属于嵌入式数据库家族。它是最广泛部署的数据库引擎，因为它被多个顶级网络浏览器、操作系统、手机和其他嵌入式系统使用。

在这个演示中，我们将创建一个简单的对话链，该链使用 `ConversationEntityMemory`，其支持的是 `SqliteEntityStore`。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

要使用存储，您只需要提供两样东西：

1. 会话 ID - 会话的唯一标识符，比如用户名、电子邮件、聊天 ID 等。

2. 连接字符串 - 一个指定数据库连接的字符串。对于 SQLite，该字符串是 `slqlite:///`，后跟数据库文件的名称。如果该文件不存在，它将被创建。

```python
from langchain_community.chat_message_histories import SQLChatMessageHistory
chat_message_history = SQLChatMessageHistory(
    session_id="test_session_id", connection_string="sqlite:///sqlite.db"
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

## 链接

我们可以轻松地将这个消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 结合起来。

为此，我们将需要使用 OpenAI，因此我们需要安装它。我们还需要将 OPENAI_API_KEY 环境变量设置为您的 OpenAI 密钥。

```bash
pip install -U langchain-openai
export OPENAI_API_KEY='sk-xxxxxxx'
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
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
    lambda session_id: SQLChatMessageHistory(
        session_id=session_id, connection_string="sqlite:///sqlite.db"
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这是我们配置会话 ID 的地方
config = {"configurable": {"session_id": "<SQL_SESSION_ID>"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```output
AIMessage(content='Hello Bob! How can I assist you today?')
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content='Your name is Bob! Is there anything specific you would like assistance with, Bob?')
```