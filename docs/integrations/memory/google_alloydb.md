# Google AlloyDB for PostgreSQL

[Google Cloud AlloyDB for PostgreSQL](https://cloud.google.com/alloydb) 是一种完全托管的 `PostgreSQL` 兼容数据库服务，适用于最苛刻的企业工作负载。`AlloyDB` 结合了 `Google Cloud` 和 `PostgreSQL` 的优势，提供卓越的性能、扩展性和可用性。通过 `AlloyDB` Langchain 集成，可以扩展数据库应用程序以构建基于人工智能的体验。

这篇笔记介绍了如何使用 `Google Cloud AlloyDB for PostgreSQL` 存储聊天消息历史记录，使用 `AlloyDBChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-alloydb-pg-python/) 上了解更多关于这个包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-alloydb-pg-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行这个笔记本，您需要完成以下步骤：

- [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 AlloyDB API](https://console.cloud.google.com/flows/enableapi?apiid=alloydb.googleapis.com)

- [创建一个 AlloyDB 实例](https://cloud.google.com/alloydb/docs/instance-primary-create)

- [创建一个 AlloyDB 数据库](https://cloud.google.com/alloydb/docs/database-create)

- [向数据库添加一个 IAM 数据库用户](https://cloud.google.com/alloydb/docs/manage-iam-authn)（可选）

### 🦜🔗 安装库

这个集成位于自己的 `langchain-google-alloydb-pg` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-alloydb-pg langchain-google-vertexai
```

**仅限 Colab：** 取消下面的注释以重新启动内核，或者使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 安装后自动重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 认证

作为在这个笔记本中登录的 IAM 用户，对 Google Cloud 进行身份验证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行这个笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[这里的设置说明](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在这个笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 参考支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 启用 API

`langchain-google-alloydb-pg` 包要求您在您的 Google Cloud 项目中[启用 AlloyDB Admin API](https://console.cloud.google.com/flows/enableapi?apiid=alloydb.googleapis.com)。

```python
# 启用 AlloyDB API
!gcloud services enable alloydb.googleapis.com
```

## 基本用法

### 设置 AlloyDB 数据库值

在[AlloyDB 集群页面](https://console.cloud.google.com/alloydb?_ga=2.223735448.2062268965.1707700487-2088871159.1707257687)中找到您的数据库值。

```python
# @title 在这里设置您的值 { display-mode: "form" }
REGION = "us-central1"  # @param {type: "string"}
CLUSTER = "my-alloydb-cluster"  # @param {type: "string"}
INSTANCE = "my-alloydb-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
```

### AlloyDBEngine 连接池

要将 AlloyDB 作为 ChatMessageHistory 内存存储库，需要一个 `AlloyDBEngine` 对象。`AlloyDBEngine` 配置了一个连接池，用于连接到您的 AlloyDB 数据库，从而实现应用程序的成功连接，并遵循行业最佳实践。

要使用 `AlloyDBEngine.from_instance()` 创建一个 `AlloyDBEngine`，您只需要提供以下 5 个参数：

1. `project_id`：AlloyDB 实例所在的 Google Cloud 项目 ID。

2. `region`：AlloyDB 实例所在的区域。

3. `cluster`：AlloyDB 集群的名称。

4. `instance`：AlloyDB 实例的名称。

5. `database`：要连接到的 AlloyDB 实例上的数据库的名称。

默认情况下，将使用[IAM数据库身份验证](https://cloud.google.com/alloydb/docs/manage-iam-authn)作为数据库身份验证的方法。该库使用从环境中获取的属于[应用程序默认凭据（ADC）](https://cloud.google.com/docs/authentication/application-default-credentials)的IAM主体。

可选地，也可以使用内置的数据库身份验证，使用用户名和密码访问AlloyDB数据库。只需向`AlloyDBEngine.from_instance()`提供可选的`user`和`password`参数：

- `user`：用于内置数据库身份验证和登录的数据库用户

- `password`：用于内置数据库身份验证和登录的数据库密码

```python
from langchain_google_alloydb_pg import AlloyDBEngine
engine = AlloyDBEngine.from_instance(
    project_id=PROJECT_ID,
    region=REGION,
    cluster=CLUSTER,
    instance=INSTANCE,
    database=DATABASE,
)
```

### 初始化表

`AlloyDBChatMessageHistory`类需要一个具有特定模式的数据库表，以存储聊天消息历史记录。

`AlloyDBEngine`引擎有一个辅助方法`init_chat_history_table()`，可用于为您创建具有适当模式的表。

```python
engine.init_chat_history_table(table_name=TABLE_NAME)
```

### AlloyDBChatMessageHistory

要初始化`AlloyDBChatMessageHistory`类，您只需要提供以下3个内容：

1. `engine` - 一个`AlloyDBEngine`引擎的实例。

2. `session_id` - 一个指定会话ID的唯一标识字符串。

3. `table_name`：要在AlloyDB数据库中存储聊天消息历史记录的表的名称。

```python
from langchain_google_alloydb_pg import AlloyDBChatMessageHistory
history = AlloyDBChatMessageHistory.create_sync(
    engine, session_id="test_session", table_name=TABLE_NAME
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

#### 清理

当特定会话的历史记录过时且可以删除时，可以按以下方式执行。

**注意：**一旦删除，数据将不再存储在AlloyDB中，将永远消失。

```python
history.clear()
```

## 🔗 链接

我们可以轻松将此消息历史记录类与[LCEL Runnables](/docs/how_to/message_history)结合使用。

为此，我们将使用[Google的Vertex AI聊天模型](/docs/integrations/chat/google_vertex_ai_palm)，这需要您在Google Cloud项目中[启用Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)。

```python
# 启用Vertex AI API
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
    lambda session_id: AlloyDBChatMessageHistory.create_sync(
        engine,
        session_id=session_id,
        table_name=TABLE_NAME,
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这是我们配置会话ID的地方
config = {"configurable": {"session_id": "test_session"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```