# 谷歌 El Carro Oracle

[Google Cloud El Carro Oracle](https://github.com/GoogleCloudPlatform/elcarro-oracle-operator) 提供了一种在 Kubernetes 中运行 `Oracle` 数据库的方式，作为一个便携、开源、社区驱动、无供应商锁定的容器编排系统。`El Carro` 提供了一个强大的声明式 API，用于全面和一致的配置和部署，以及实时操作和监控。通过利用 `El Carro` Langchain 集成，可以扩展您的 `Oracle` 数据库的功能，以构建基于人工智能的体验。

本指南介绍了如何使用 `El Carro` Langchain 集成来使用 `ElCarroChatMessageHistory` 类存储聊天消息历史记录。无论 `Oracle` 数据库在何处运行，此集成都适用。

在 [GitHub](https://github.com/googleapis/langchain-google-el-carro-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-el-carro-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要完成以下步骤：

* 如果您想要使用 El Carro 运行您的 Oracle 数据库，请完成 [入门](https://github.com/googleapis/langchain-google-el-carro-python/tree/main/README.md#getting-started) 部分。

### 🦜🔗 库安装

该集成位于其自己的 `langchain-google-el-carro` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-el-carro langchain-google-vertexai langchain
```

**仅适用于 Colab：** 取消下面的注释以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动安装后重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 身份验证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 身份验证，以便访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

* 如果您使用 Vertex AI Workbench，请查看此处的设置说明 [here](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
# from google.colab import auth
# auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 参阅支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

## 基本用法

### 设置 Oracle 数据库连接

填写以下变量，使用您的 Oracle 数据库连接详细信息。

```python
HOST = "127.0.0.1"  # @param {type: "string"}
PORT = 3307  # @param {type: "integer"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
USER = "my-user"  # @param {type: "string"}
PASSWORD = input("请提供用于数据库用户的密码：")
```

如果您使用 `El Carro`，您可以在 `El Carro` Kubernetes 实例的状态中找到主机名和端口值。

### ElCarroEngine 连接池

`ElCarroEngine` 配置了一个连接池，用于连接到您的 Oracle 数据库，从而使您的应用程序能够成功连接并遵循行业最佳实践。

```python
from langchain_google_el_carro import ElCarroEngine
elcarro_engine = ElCarroEngine.from_instance(
    db_host=HOST,
    db_port=PORT,
    db_name=DATABASE,
    db_user=USER,
    db_password=PASSWORD,
)
```

### 初始化表

`ElCarroChatMessageHistory` 类需要一个具有特定模式的数据库表，以存储聊天消息历史记录。

`ElCarroEngine` 类有一个 `init_chat_history_table()` 方法，可以用于为您创建具有适当模式的表。

```python
elcarro_engine.init_chat_history_table(table_name=TABLE_NAME)
```

### ElCarroChatMessageHistory

要初始化 `ElCarroChatMessageHistory` 类，您只需要提供以下 3 个要素：

1. `elcarro_engine` - 一个 `ElCarroEngine` 引擎的实例。

2. `session_id` - 一个唯一标识字符串，用于指定会话的 id。

3. `table_name`：要在 Oracle 数据库中存储聊天消息历史记录的表的名称。

```python
from langchain_google_el_carro import ElCarroChatMessageHistory
history = ElCarroChatMessageHistory(
    elcarro_engine=elcarro_engine, session_id="test_session", table_name=TABLE_NAME
)
history.add_user_message("hi!")
history.add_ai_message("whats up!")
```

```python
history.messages
```

#### 清理

当特定会话的历史记录过时并且可以被删除时，可以按以下方式进行操作。

**注意：** 一旦删除，数据将不再存储在您的数据库中，将永远消失。

```python
history.clear()
```

## 🔗 链接

我们可以轻松地将此消息历史记录类与[LCEL Runnables](/docs/how_to/message_history)结合使用。

为此，我们将使用[Google 的 Vertex AI 聊天模型](/docs/integrations/chat/google_vertex_ai_palm)，该模型要求您在 Google Cloud 项目中[启用 Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)。

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
    lambda session_id: ElCarroChatMessageHistory(
        elcarro_engine,
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

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```