# Postgres

[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL)（也被称为 `Postgres`）是一个强调可扩展性和SQL兼容性的免费开源关系数据库管理系统（RDBMS）。

本文介绍如何使用Postgres存储聊天消息记录。

```python
from langchain_community.chat_message_histories import (
    PostgresChatMessageHistory,
)
history = PostgresChatMessageHistory(
    connection_string="postgresql://postgres:mypassword@localhost/chat_history",
    session_id="foo",
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```