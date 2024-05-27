# TiDB

> [TiDB Cloud](https://tidbcloud.com/) 是一种全面的数据库即服务（DBaaS）解决方案，提供了专用和无服务器选项。TiDB 无服务器现在正在将内置的向量搜索集成到 MySQL 环境中。通过这一增强功能，您可以在 TiDB 无服务器上无缝开发人工智能应用，无需新建数据库或额外的技术堆栈。立即加入私人测试版的等候名单，成为第一批体验者，网址为 https://tidb.cloud/ai。

本笔记本介绍了如何使用 TiDB 存储聊天消息历史记录。

## 设置

首先，我们将安装以下依赖项：

```python
%pip install --upgrade --quiet langchain langchain_openai
```

配置您的 OpenAI 密钥

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("输入您的 OpenAI API 密钥：")
```

最后，我们将配置连接到 TiDB。在本笔记本中，我们将按照 TiDB Cloud 提供的标准连接方法建立安全高效的数据库连接。

```python
# 从 TiDB Cloud 控制台复制
tidb_connection_string_template = "mysql+pymysql://<USER>:<PASSWORD>@<HOST>:4000/<DB>?ssl_ca=/etc/ssl/cert.pem&ssl_verify_cert=true&ssl_verify_identity=true"
tidb_password = getpass.getpass("输入您的 TiDB 密码：")
tidb_connection_string = tidb_connection_string_template.replace(
    "<PASSWORD>", tidb_password
)
```

## 生成历史数据

创建一组历史数据，这将成为我们即将进行的演示的基础。

```python
from datetime import datetime
from langchain_community.chat_message_histories import TiDBChatMessageHistory
history = TiDBChatMessageHistory(
    connection_string=tidb_connection_string,
    session_id="code_gen",
    earliest_time=datetime.utcnow(),  # 可选，设置 earliest_time 以加载此时间点之后的消息。
)
history.add_user_message("我们的功能进展如何？")
history.add_ai_message(
    "进展顺利。我们正在进行测试。将在二月发布。"
)
```

```python
history.messages
```

```output
[HumanMessage(content="我们的功能进展如何？"),
 AIMessage(content="进展顺利。我们正在进行测试。将在二月发布。")]
```

## 与历史数据交谈

让我们基于之前生成的历史数据，创建一个动态的聊天交互。

首先，使用 LangChain 创建一个聊天链：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "您是一个擅长编码的助手，正在帮助一家初创公司构建",
        ),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
```

构建一个带有历史记录的可运行对象：

```python
from langchain_core.runnables.history import RunnableWithMessageHistory
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: TiDBChatMessageHistory(
        session_id=session_id, connection_string=tidb_connection_string
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

启动聊天：

```python
response = chain_with_history.invoke(
    {"question": "今天是一月一日。我们的功能还有多少天才能发布？"},
    config={"configurable": {"session_id": "code_gen"}},
)
response
```

```output
AIMessage(content='一月有 31 天，所以距离我们的功能在二月发布还有 30 天。')
```

## 检查历史数据

```python
history.reload_cache()
history.messages
```

```output
[HumanMessage(content="我们的功能进展如何？"),
 AIMessage(content="进展顺利。我们正在进行测试。将在二月发布。"),
 HumanMessage(content='今天是一月一日。我们的功能还有多少天才能发布？'),
 AIMessage(content='一月有 31 天，所以距离我们的功能在二月发布还有 30 天。')]
```