# 谷歌 Spanner

> [Google Cloud Spanner](https://cloud.google.com/spanner) 是一个高度可扩展的数据库，它将无限可扩展性与关系语义（如次要索引、强一致性、模式和 SQL）结合在一个简单的解决方案中，提供 99.999% 的可用性。

本笔记本介绍了如何使用 `Spanner` 存储聊天消息历史，使用 `SpannerChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-spanner-python/) 上了解更多关于这个包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-spanner-python/blob/main/samples/chat_message_history.ipynb)

## 开始之前

要运行这个笔记本，您需要执行以下操作：

- [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Cloud Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)

- [创建一个 Spanner 实例](https://cloud.google.com/spanner/docs/create-manage-instances)

- [创建一个 Spanner 数据库](https://cloud.google.com/spanner/docs/create-manage-databases)

### 🦜🔗 安装库

集成位于自己的 `langchain-google-spanner` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-spanner
```

**仅限 Colab：** 取消下面的注释以重新启动内核，或者使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动安装后重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 认证

作为 IAM 用户登录到这个笔记本中，需要对 Google Cloud 进行身份验证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行这个笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在这个笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 启用 API

`langchain-google-spanner` 包要求您在您的 Google Cloud 项目中[启用 Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)。

```python
# 启用 Spanner API
!gcloud services enable spanner.googleapis.com
```

## 基本用法

### 设置 Spanner 数据库值

在 [Spanner 实例页面](https://console.cloud.google.com/spanner)中找到您的数据库值。

```python
# @title 在此处设置您的值 { display-mode: "form" }
INSTANCE = "my-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
```

### 初始化表

`SpannerChatMessageHistory` 类需要一个具有特定模式的数据库表，以存储聊天消息历史记录。

可以使用辅助方法 `init_chat_history_table()` 来为您创建具有适当模式的表。

```python
from langchain_google_spanner import (
    SpannerChatMessageHistory,
)
SpannerChatMessageHistory.init_chat_history_table(table_name=TABLE_NAME)
```

### SpannerChatMessageHistory

要初始化 `SpannerChatMessageHistory` 类，您只需要提供 3 个东西：

1. `instance_id` - Spanner 实例的名称

2. `database_id` - Spanner 数据库的名称

3. `session_id` - 一个唯一标识符字符串，用于指定会话的 id

4. `table_name` - 数据库中存储聊天消息历史记录的表的名称

```python
message_history = SpannerChatMessageHistory(
    instance_id=INSTANCE,
    database_id=DATABASE,
    table_name=TABLE_NAME,
    session_id="user-session-id",
)
message_history.add_user_message("hi!")
message_history.add_ai_message("whats up?")
```

```python
message_history.messages
```

## 自定义客户端

默认情况下创建的客户端是默认客户端。要使用非默认客户端，可以将[自定义客户端](https://cloud.google.com/spanner/docs/samples/spanner-create-client-with-query-options#spanner_create_client_with_query_options-python)传递给构造函数。

```python
from google.cloud import spanner
custom_client_message_history = SpannerChatMessageHistory(
    instance_id="my-instance",
    database_id="my-database",
    client=spanner.Client(...),
)
```

## 清理

当特定会话的历史记录过时且可以被删除时，可以按以下方式进行操作。

注意：一旦删除，数据将不再存储在 Cloud Spanner 中，将永久丢失。

```python
message_history = SpannerChatMessageHistory(
    instance_id=INSTANCE,
    database_id=DATABASE,
    table_name=TABLE_NAME,
    session_id="user-session-id",
)
message_history.clear()
```