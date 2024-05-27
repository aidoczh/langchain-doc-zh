# Neo4j

[Neo4j](https://zh.wikipedia.org/wiki/Neo4j) 是一个开源的图数据库管理系统，以其高效管理高度连接数据而闻名。与传统的以表格形式存储数据的数据库不同，Neo4j 使用节点、边和属性的图结构来表示和存储数据。这种设计允许对复杂数据关系进行高性能查询。

本文介绍如何使用 `Neo4j` 存储聊天消息历史记录。

```python
from langchain_community.chat_message_histories import Neo4jChatMessageHistory
history = Neo4jChatMessageHistory(
    url="bolt://localhost:7687",
    username="neo4j",
    password="password",
    session_id="session_id_1",
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```