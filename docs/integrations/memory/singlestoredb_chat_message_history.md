# SingleStoreDB

本文介绍了如何使用 SingleStoreDB 存储聊天消息历史记录。

```python
from langchain_community.chat_message_histories import (
    SingleStoreDBChatMessageHistory,
)
history = SingleStoreDBChatMessageHistory(
    session_id="foo", host="root:pass@localhost:3306/db"
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```