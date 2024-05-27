# 适用于 SQL Server 的 Google SQL

[Google Cloud SQL](https://cloud.google.com/sql) 是一个完全托管的关系型数据库服务，提供高性能、无缝集成和令人印象深刻的可扩展性。它提供 `MySQL`、`PostgreSQL` 和 `SQL Server` 数据库引擎。通过 Cloud SQL 的 Langchain 集成，可以扩展数据库应用程序以构建基于人工智能的体验。

本笔记本介绍了如何使用 `Google Cloud SQL for SQL Server` 来存储聊天消息历史，使用 `MSSQLChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-cloud-sql-mssql-python/) 上了解有关该软件包的更多信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-cloud-sql-mssql-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

- [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Cloud SQL Admin API](https://console.cloud.google.com/marketplace/product/google/sqladmin.googleapis.com)

- [创建一个适用于 SQL Server 的 Cloud SQL 实例](https://cloud.google.com/sql/docs/sqlserver/create-instance)

- [创建一个 Cloud SQL 数据库](https://cloud.google.com/sql/docs/sqlserver/create-manage-databases)

- [创建一个数据库用户](https://cloud.google.com/sql/docs/sqlserver/create-manage-users)（如果选择使用 `sqlserver` 用户，则可选）

### 🦜🔗 安装库

集成位于自己的 `langchain-google-cloud-sql-mssql` 软件包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-cloud-sql-mssql langchain-google-vertexai
```

**仅适用于 Colab：**取消下面的注释以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 认证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 认证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[此处的设置说明](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 参见支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 API 启用

`langchain-google-cloud-sql-mssql` 软件包要求您在 Google Cloud 项目中[启用 Cloud SQL Admin API](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin.googleapis.com)。

```python
# 启用 Cloud SQL Admin API
!gcloud services enable sqladmin.googleapis.com
```

## 基本用法

### 设置 Cloud SQL 数据库值

在[Cloud SQL 实例页面](https://console.cloud.google.com/sql?_ga=2.223735448.2062268965.1707700487-2088871159.1707257687)中找到您的数据库值。

```python
# @title 在此处设置您的值 { display-mode: "form" }
REGION = "us-central1"  # @param {type: "string"}
INSTANCE = "my-mssql-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
DB_USER = "my-username"  # @param {type: "string"}
DB_PASS = "my-password"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
```

### MSSQLEngine 连接池

要将 Cloud SQL 设置为 ChatMessageHistory 内存存储的要求和参数之一是 `MSSQLEngine` 对象。`MSSQLEngine` 配置了连接池到您的 Cloud SQL 数据库，使您的应用程序能够成功连接，并遵循行业最佳实践。

要使用 `MSSQLEngine.from_instance()` 创建 `MSSQLEngine`，您只需要提供 6 个参数：

1. `project_id`：Cloud SQL 实例所在的 Google Cloud 项目 ID。

2. `region`：Cloud SQL 实例所在的区域。

3. `instance`：Cloud SQL 实例的名称。

4. `database`：要连接到的 Cloud SQL 实例上的数据库的名称。

1. `user`：用于内置数据库身份验证和登录的数据库用户。

1. `password`：用于内置数据库身份验证和登录的数据库密码。

默认情况下，使用用户名和密码进行内置数据库身份验证来访问 Cloud SQL 数据库。

```python
from langchain_google_cloud_sql_mssql import MSSQLEngine
engine = MSSQLEngine.from_instance(
    project_id=PROJECT_ID,
    region=REGION,
    instance=INSTANCE,
    database=DATABASE,
    user=DB_USER,
    password=DB_PASS,
)
```

### 初始化表格

`MSSQLChatMessageHistory` 类需要一个具有特定模式的数据库表格，以便存储聊天消息历史记录。

`MSSQLEngine` 引擎有一个辅助方法 `init_chat_history_table()`，可以用来为您创建具有正确模式的表格。

```python
engine.init_chat_history_table(table_name=TABLE_NAME)
```

### MSSQLChatMessageHistory

要初始化 `MSSQLChatMessageHistory` 类，您只需要提供以下 3 个内容：

1. `engine` - `MSSQLEngine` 引擎的一个实例。

1. `session_id` - 一个唯一的标识字符串，用于指定会话的 id。

1. `table_name`：存储聊天消息历史记录的 Cloud SQL 数据库中的表格名称。

```python
from langchain_google_cloud_sql_mssql import MSSQLChatMessageHistory
history = MSSQLChatMessageHistory(
    engine, session_id="test_session", table_name=TABLE_NAME
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

```output
[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```

#### 清理

当特定会话的历史记录过时且可以删除时，可以按以下方式进行删除。

**注意：**一旦删除，数据将不再存储在 Cloud SQL 中，将永远丢失。

```python
history.clear()
```

## 🔗 链接

我们可以轻松将此消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 结合使用。

为此，我们将使用其中之一的 [Google 的 Vertex AI 聊天模型](/docs/integrations/chat/google_vertex_ai_palm)，该模型要求您在 Google Cloud 项目中 [启用 Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)。

```python
# 启用 Vertex AI API
!gcloud services enable aiplatform.googleapis.com
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_google_vertexai import ChatVertexAI
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatVertexAI(project=PROJECT_ID)
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: MSSQLChatMessageHistory(
        engine,
        session_id=session_id,
        table_name=TABLE_NAME,
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这是我们配置会话 id 的地方
config = {"configurable": {"session_id": "test_session"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```output
AIMessage(content=' Hello Bob, how can I help you today?')
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content=' Your name is Bob.')
```