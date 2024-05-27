# Astra DB 

> DataStax 的 [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 是建立在Cassandra之上的无服务器矢量数据库，通过易于使用的JSON API方便地提供。

本笔记将介绍如何使用Astra DB存储聊天消息记录。

## 设置

要运行此笔记，您需要一个正在运行的Astra DB。在Astra仪表板上获取连接密钥：

- API端点看起来像 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`;

- 令牌看起来像 `AstraCS:6gBhNmsk135...`。

```python
%pip install --upgrade --quiet  "astrapy>=0.7.1"
```

### 设置数据库连接参数和密钥

```python
import getpass
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass.getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```

```output
ASTRA_DB_API_ENDPOINT =  https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com
ASTRA_DB_APPLICATION_TOKEN =  ········
```

根据是本地还是基于云的Astra DB，创建相应的数据库连接“Session”对象。

## 示例

```python
from langchain_community.chat_message_histories import AstraDBChatMessageHistory
message_history = AstraDBChatMessageHistory(
    session_id="test-session",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
)
message_history.add_user_message("hi!")
message_history.add_ai_message("whats up?")
```

```python
message_history.messages
```

```output
[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```