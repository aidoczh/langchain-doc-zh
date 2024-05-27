# Momento 缓存

[Momento 缓存](https://docs.momentohq.com/) 是世界上第一个真正的无服务器缓存服务。它提供即时的弹性、零缩放能力和极快的性能。

本笔记介绍如何使用 [Momento 缓存](https://www.gomomento.com/services/cache) 来存储聊天消息历史，使用 `MomentoChatMessageHistory` 类。有关如何设置 Momento 的更多详细信息，请参阅 Momento 的[文档](https://docs.momentohq.com/getting-started)。

请注意，默认情况下，如果不存在具有给定名称的缓存，我们将创建一个缓存。

您需要获取 Momento API 密钥才能使用这个类。这可以直接传递给 `MomentoChatMessageHistory.from_client_params`，或者可以设置为环境变量 `MOMENTO_API_KEY`。

```python
from datetime import timedelta
from langchain_community.chat_message_histories import MomentoChatMessageHistory
session_id = "foo"
cache_name = "langchain"
ttl = timedelta(days=1)
history = MomentoChatMessageHistory.from_client_params(
    session_id,
    cache_name,
    ttl,
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

```output
[HumanMessage(content='hi!', additional_kwargs={}, example=False),
 AIMessage(content='whats up?', additional_kwargs={}, example=False)]
```