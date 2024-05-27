# Redis

>[Redis (Remote Dictionary Server)](https://en.wikipedia.org/wiki/Redis) 是一个开源的内存存储系统，用作分布式内存键值数据库、缓存和消息代理，可选择性地具备持久性。由于它将所有数据存储在内存中，并且由于其设计，`Redis` 提供了低延迟的读写，使其特别适用于需要缓存的用例。Redis 是最流行的 NoSQL 数据库之一，也是最受欢迎的数据库之一。

本笔记将介绍如何使用 `Redis` 存储聊天消息历史记录。

## 设置

首先，我们需要安装依赖项，并使用类似 `redis-server` 的命令启动一个 Redis 实例。

```python
pip install -U langchain-community redis
```

```python
from langchain_community.chat_message_histories import RedisChatMessageHistory
```

## 存储和检索消息

```python
history = RedisChatMessageHistory("foo", url="redis://localhost:6379")
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

```output
[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```

## 在链中使用

```python
pip install -U langchain-openai
```

```python
from typing import Optional
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You're an assistant。"),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: RedisChatMessageHistory(
        session_id, url="redis://localhost:6379"
    ),
    input_messages_key="question",
    history_messages_key="history",
)
config = {"configurable": {"session_id": "foo"}}
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content='Your name is Bob, as you mentioned earlier. Is there anything specific you would like assistance with, Bob?')
```