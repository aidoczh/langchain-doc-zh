# Google SQL for PostgreSQL

> [Google Cloud SQL](https://cloud.google.com/sql) 是一个完全托管的关系型数据库服务，提供高性能、无缝集成和可扩展性强的特点。它支持 `MySQL`、`PostgreSQL` 和 `SQL Server` 数据库引擎。通过 Cloud SQL 的 Langchain 集成，您可以扩展数据库应用程序，构建基于 AI 的体验。

本文介绍如何使用 `Google Cloud SQL for PostgreSQL` 存储聊天消息历史记录，使用 `PostgresChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-cloud-sql-pg-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-cloud-sql-pg-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行这个笔记本，您需要完成以下步骤：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

* [启用 Cloud SQL 管理 API](https://console.cloud.google.com/marketplace/product/google/sqladmin.googleapis.com)

* [创建一个 Cloud SQL for PostgreSQL 实例](https://cloud.google.com/sql/docs/postgres/create-instance)

* [创建一个 Cloud SQL 数据库](https://cloud.google.com/sql/docs/mysql/create-manage-databases)

* [向数据库添加一个 IAM 数据库用户](https://cloud.google.com/sql/docs/postgres/add-manage-iam-users#creating-a-database-user)（可选）

### 🦜🔗 安装库

该集成功能位于自己的 `langchain-google-cloud-sql-pg` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-cloud-sql-pg langchain-google-vertexai
```

**仅限 Colab：**取消注释以下单元格以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 身份验证

以作为此笔记本中登录的 IAM 用户的身份验证到 Google Cloud，以便访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

* 如果您使用 Vertex AI Workbench，请查看[此处的设置说明](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行该单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 启用 API

`langchain-google-cloud-sql-pg` 包要求您在 Google Cloud 项目中[启用 Cloud SQL 管理 API](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin.googleapis.com)。

```python
# 启用 Cloud SQL 管理 API
!gcloud services enable sqladmin.googleapis.com
```

## 基本用法

### 设置 Cloud SQL 数据库值

在 [Cloud SQL 实例页面](https://console.cloud.google.com/sql?_ga=2.223735448.2062268965.1707700487-2088871159.1707257687)中找到您的数据库值。

```python
# @title 在此处设置您的值 { display-mode: "form" }
REGION = "us-central1"  # @param {type: "string"}
INSTANCE = "my-postgresql-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
```

### PostgresEngine 连接池

将 Cloud SQL 作为 ChatMessageHistory 内存存储的要求和参数之一是一个 `PostgresEngine` 对象。`PostgresEngine` 配置了一个连接池到您的 Cloud SQL 数据库，使您的应用程序能够成功连接，并遵循行业最佳实践。

要使用 `PostgresEngine.from_instance()` 创建一个 `PostgresEngine`，您只需要提供以下 4 个参数：

1. `project_id`：Cloud SQL 实例所在的 Google Cloud 项目的项目 ID。

2. `region`：Cloud SQL 实例所在的区域。

3. `instance`：Cloud SQL 实例的名称。

4. `database`：要连接的 Cloud SQL 实例上的数据库的名称。

默认情况下，将使用 [IAM 数据库身份验证](https://cloud.google.com/sql/docs/postgres/iam-authentication#iam-db-auth) 作为数据库身份验证的方法。此库使用从环境中获取的属于 [应用程序默认凭据 (ADC)](https://cloud.google.com/docs/authentication/application-default-credentials) 的 IAM 主体。

有关 IAM 数据库身份验证的更多信息，请参见：

- [配置 IAM 数据库身份验证的实例](https://cloud.google.com/sql/docs/postgres/create-edit-iam-instances)

- [使用 IAM 数据库身份验证管理用户](https://cloud.google.com/sql/docs/postgres/add-manage-iam-users)

还可以选择使用 [内置数据库身份验证](https://cloud.google.com/sql/docs/postgres/built-in-authentication)，使用用户名和密码访问 Cloud SQL 数据库。只需向 `PostgresEngine.from_instance()` 提供可选的 `user` 和 `password` 参数：

- `user`：用于内置数据库身份验证和登录的数据库用户

- `password`：用于内置数据库身份验证和登录的数据库密码

```python
from langchain_google_cloud_sql_pg import PostgresEngine
engine = PostgresEngine.from_instance(
    project_id=PROJECT_ID, region=REGION, instance=INSTANCE, database=DATABASE
)
```

### 初始化表

`PostgresChatMessageHistory` 类需要具有特定模式的数据库表，以存储聊天消息历史记录。

`PostgresEngine` 引擎具有一个辅助方法 `init_chat_history_table()`，可用于为您创建具有适当模式的表。

```python
engine.init_chat_history_table(table_name=TABLE_NAME)
```

### PostgresChatMessageHistory

要初始化 `PostgresChatMessageHistory` 类，您只需要提供以下 3 个内容：

1. `engine` - `PostgresEngine` 引擎的实例。

2. `session_id` - 一个唯一标识符字符串，指定会话的 id。

3. `table_name`：要在 Cloud SQL 数据库中存储聊天消息历史记录的表的名称。

```python
from langchain_google_cloud_sql_pg import PostgresChatMessageHistory
history = PostgresChatMessageHistory.create_sync(
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

当特定会话的历史记录过时且可以被删除时，可以按以下方式执行。

**注意：**一旦删除，数据将不再存储在 Cloud SQL 中，将永远丢失。

```python
history.clear()
```

## 🔗 链接

我们可以轻松地将此消息历史记录类与 [LCEL Runnables](/docs/how_to/message_history) 结合使用。

为此，我们将使用 [Google 的 Vertex AI 聊天模型](/docs/integrations/chat/google_vertex_ai_palm) 之一，该模型要求您在 Google Cloud 项目中 [启用 Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)。

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
    lambda session_id: PostgresChatMessageHistory.create_sync(
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