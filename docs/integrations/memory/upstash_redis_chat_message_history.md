# Upstash Redis

[Upstash](https://upstash.com/docs/introduction) 是一个提供无服务器 `Redis`、`Kafka` 和 `QStash` API 的服务商。

这篇笔记介绍了如何使用 `Upstash Redis` 存储聊天消息历史记录。

```python
from langchain_community.chat_message_histories import (
    UpstashRedisChatMessageHistory,
)
URL = "<UPSTASH_REDIS_REST_URL>"
TOKEN = "<UPSTASH_REDIS_REST_TOKEN>"
history = UpstashRedisChatMessageHistory(
    url=URL, token=TOKEN, ttl=10, session_id="my-test-session"
)
history.add_user_message("hello llm!")
history.add_ai_message("hello user!")
```

```python
history.messages
```