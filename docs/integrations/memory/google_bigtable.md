# 谷歌 Bigtable

[Google Cloud Bigtable](https://cloud.google.com/bigtable) 是一种键-值和宽列存储，非常适合快速访问结构化、半结构化或非结构化数据。通过 Bigtable 的 Langchain 集成，扩展数据库应用程序以构建利用人工智能的体验。

这个笔记本将介绍如何使用 [Google Cloud Bigtable](https://cloud.google.com/bigtable) 存储聊天消息历史，使用 `BigtableChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-bigtable-python/) 上了解更多关于这个包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-bigtable-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行这个笔记本，您需要完成以下步骤：

- [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Bigtable API](https://console.cloud.google.com/flows/enableapi?apiid=bigtable.googleapis.com)

- [创建一个 Bigtable 实例](https://cloud.google.com/bigtable/docs/creating-instance)

- [创建一个 Bigtable 表](https://cloud.google.com/bigtable/docs/managing-tables)

- [创建 Bigtable 访问凭证](https://developers.google.com/workspace/guides/create-credentials)

### 🦜🔗 库安装

集成位于自己的 `langchain-google-bigtable` 包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-bigtable
```

**仅适用于 Colab**：取消下面的代码注释以重新启动内核，或者使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 安装后自动重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填入您的 Google Cloud 项目 ID，然后运行该单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 🔐 身份验证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 身份验证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### 初始化 Bigtable 模式

BigtableChatMessageHistory 的模式需要实例和表存在，并且有一个名为 `langchain` 的列族。

```python
# @markdown 请为演示目的指定一个实例和一个表。
INSTANCE_ID = "my_instance"  # @param {type:"string"}
TABLE_ID = "my_table"  # @param {type:"string"}
```

如果表或列族不存在，您可以使用以下函数来创建它们。

```python
from google.cloud import bigtable
from langchain_google_bigtable import create_chat_history_table
create_chat_history_table(
    instance_id=INSTANCE_ID,
    table_id=TABLE_ID,
)
```

### BigtableChatMessageHistory

要初始化 `BigtableChatMessageHistory` 类，您只需要提供以下 3 个内容：

1. `instance_id` - 用于聊天消息历史记录的 Bigtable 实例。

2. `table_id`：用于存储聊天消息历史记录的 Bigtable 表。

3. `session_id` - 一个唯一标识符字符串，指定会话的 id。

```python
from langchain_google_bigtable import BigtableChatMessageHistory
message_history = BigtableChatMessageHistory(
    instance_id=INSTANCE_ID,
    table_id=TABLE_ID,
    session_id="user-session-id",
)
message_history.add_user_message("hi!")
message_history.add_ai_message("whats up?")
```

```python
message_history.messages
```

#### 清理

当特定会话的历史记录过时且可以删除时，可以按以下方式执行。

**注意：**一旦删除，数据将不再存储在 Bigtable 中，将永远消失。

```python
message_history.clear()
```

## 高级用法

### 自定义客户端

默认情况下创建的客户端是默认客户端，只使用 admin=True 选项。要使用非默认客户端，可以将 [自定义客户端](https://cloud.google.com/python/docs/reference/bigtable/latest/client#class-googlecloudbigtableclientclientprojectnone-credentialsnone-readonlyfalse-adminfalse-clientinfonone-clientoptionsnone-adminclientoptionsnone-channelnone) 传递给构造函数。

```python
from google.cloud import bigtable
client = (bigtable.Client(...),)
create_chat_history_table(
    instance_id="my-instance",
    table_id="my-table",
    client=client,
)
custom_client_message_history = BigtableChatMessageHistory(
    instance_id="my-instance",
    table_id="my-table",
    client=client,
)
```