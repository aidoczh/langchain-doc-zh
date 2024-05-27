# Google Firestore（原生模式）

> [Google Cloud Firestore](https://cloud.google.com/firestore) 是一个无服务器的面向文档的数据库，可以根据需求进行扩展。通过使用 `Firestore` 的 Langchain 集成，可以扩展数据库应用程序以构建基于人工智能的体验。

本文介绍了如何使用 [Google Cloud Firestore](https://cloud.google.com/firestore) 来存储聊天消息历史记录，使用 `FirestoreChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-firestore-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-firestore-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

* [创建 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

* [启用 Firestore API](https://console.cloud.google.com/flows/enableapi?apiid=firestore.googleapis.com)

* [创建 Firestore 数据库](https://cloud.google.com/firestore/docs/manage-databases)

在此笔记本的运行环境中确认访问数据库后，在运行示例脚本之前，填写以下值并运行单元格。

### 🦜🔗 安装库

该集成位于其自己的 `langchain-google-firestore` 包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-firestore
```

**仅限 Colab**：取消注释以下单元格以重新启动内核，或使用按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
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

### 🔐 身份验证

以登录到此笔记本的 IAM 用户的身份对 Google Cloud 进行身份验证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看此处的设置说明 [here](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### FirestoreChatMessageHistory

要初始化 `FirestoreChatMessageHistory` 类，您只需要提供 3 个参数：

1. `session_id` - 一个唯一的标识符字符串，用于指定会话的 id。

1. `collection`：一个以 `/` 分隔的 Firestore 集合的路径。

```python
from langchain_google_firestore import FirestoreChatMessageHistory
chat_history = FirestoreChatMessageHistory(
    session_id="user-session-id", collection="HistoryMessages"
)
chat_history.add_user_message("Hi!")
chat_history.add_ai_message("How can I help you?")
```

```python
chat_history.messages
```

#### 清理

当特定会话的历史记录过时并且可以从数据库和内存中删除时，可以按以下方式进行清理。

**注意：**一旦删除，数据将不再存储在 Firestore 中，将永远消失。

```python
chat_history.clear()
```

### 自定义客户端

默认情况下，客户端使用可用的环境变量创建。可以将 [自定义客户端](https://cloud.google.com/python/docs/reference/firestore/latest/client) 传递给构造函数。

```python
from google.auth import compute_engine
from google.cloud import firestore
client = firestore.Client(
    project="project-custom",
    database="non-default-database",
    credentials=compute_engine.Credentials(),
)
history = FirestoreChatMessageHistory(
    session_id="session-id", collection="History", client=client
)
history.add_user_message("New message")
history.messages
history.clear()
```
