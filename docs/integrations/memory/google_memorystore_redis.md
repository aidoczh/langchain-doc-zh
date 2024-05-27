# Google Memorystore for Redis

> [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 是一个完全托管的服务，由 Redis 内存数据存储支持，用于构建应用程序缓存，提供亚毫秒级的数据访问。通过使用 Memorystore for Redis 的 Langchain 集成，扩展您的数据库应用程序以构建基于人工智能的体验。

本笔记本介绍如何使用 [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 存储聊天消息历史记录，使用 `MemorystoreChatMessageHistory` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-memorystore-redis-python/) 上了解更多关于该软件包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-memorystore-redis-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

* [启用 Memorystore for Redis API](https://console.cloud.google.com/flows/enableapi?apiid=redis.googleapis.com)

* [创建一个 Memorystore for Redis 实例](https://cloud.google.com/memorystore/docs/redis/create-instance-console)。确保版本大于或等于 5.0。

在确认在此笔记本的运行时环境中可以访问数据库后，在运行示例脚本之前，填写以下值并运行单元格。

```python
# @markdown 请指定与实例关联的端点或演示目的。
ENDPOINT = "redis://127.0.0.1:6379"  # @param {type:"string"}
```

### 🦜🔗 安装库

该集成位于自己的 `langchain-google-memorystore-redis` 软件包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-memorystore-redis
```

**仅限 Colab：**取消注释以下单元格以重新启动内核，或使用按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动在安装后重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 参见支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行该单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 🔐 身份验证

作为登录到此笔记本的 IAM 用户，通过身份验证以访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

* 如果您使用 Vertex AI Workbench，请查看[此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### MemorystoreChatMessageHistory

要初始化 `MemorystoreMessageHistory` 类，您只需要提供 2 个参数：

1. `redis_client` - Memorystore Redis 的实例。

1. `session_id` - 每个聊天消息历史记录对象必须具有唯一的会话 ID。如果会话 ID 中已经存储了消息，则可以检索到这些消息。

```python
import redis
from langchain_google_memorystore_redis import MemorystoreChatMessageHistory
# 连接到 Memorystore for Redis 实例
redis_client = redis.from_url("redis://127.0.0.1:6379")
message_history = MemorystoreChatMessageHistory(redis_client, session_id="session1")
```

```python
message_history.messages
```

#### 清理

当特定会话的历史记录已过时且可以删除时，可以按以下方式进行删除。

**注意：**一旦删除，数据将不再存储在 Memorystore for Redis 中，将永远丢失。

```python
message_history.clear()
```