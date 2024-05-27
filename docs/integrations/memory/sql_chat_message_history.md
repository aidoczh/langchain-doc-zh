# SQL (SQLAlchemy)

[结构化查询语言 (SQL)](https://en.wikipedia.org/wiki/SQL) 是一种领域特定语言，用于编程并设计用于管理关系数据库管理系统 (RDBMS) 中保存的数据，或者用于关系数据流管理系统 (RDSMS) 中的流处理。它在处理结构化数据，即包含实体和变量之间关系的数据方面特别有用。

[SQLAlchemy](https://github.com/sqlalchemy/sqlalchemy) 是一个开源的 `SQL` 工具包和对象关系映射器 (ORM)，适用于 Python 编程语言，采用 MIT 许可证发布。

这个笔记本介绍了一个 `SQLChatMessageHistory` 类，允许在 `SQLAlchemy` 支持的任何数据库中存储聊天历史记录。

请注意，要将其与除 `SQLite` 外的数据库一起使用，您需要安装相应的数据库驱动程序。

## 设置

集成位于 `langchain-community` 包中，因此我们需要安装该包。我们还需要安装 `SQLAlchemy` 包。

```bash
pip install -U langchain-community SQLAlchemy langchain-openai
```

设置 [LangSmith](https://smith.langchain.com/) 也很有帮助（但不是必需的）以获得最佳的可观察性

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

要使用存储，您只需要提供两样东西：

1. 会话 ID - 会话的唯一标识符，如用户名、电子邮件、聊天 ID 等。

2. 连接字符串 - 指定数据库连接的字符串。它将传递给 SQLAlchemy 的 create_engine 函数。

```python
from langchain_community.chat_message_histories import SQLChatMessageHistory
chat_message_history = SQLChatMessageHistory(
    session_id="test_session", connection_string="sqlite:///sqlite.db"
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

我们可以轻松地将这个消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 结合起来

为此，我们将需要使用 OpenAI，因此我们需要安装它

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
config = {"configurable": {"session_id": "<SESSION_ID>"}}
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