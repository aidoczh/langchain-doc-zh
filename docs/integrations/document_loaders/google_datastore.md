# Google Firestore 数据库模式中的 Datastore

[Firestore 数据库模式中的 Datastore](https://cloud.google.com/datastore) 是一个专为自动扩展、高性能和应用开发便利性而构建的 NoSQL 文档数据库。通过 Datastore 的 Langchain 集成，您可以扩展数据库应用程序，构建利用人工智能的体验。

本笔记将介绍如何使用[Firestore 数据库模式中的 Datastore](https://cloud.google.com/datastore)来使用 `DatastoreLoader` 和 `DatastoreSaver` [保存、加载和删除 langchain 文档](/docs/how_to#document-loaders)。

在 [GitHub](https://github.com/googleapis/langchain-google-datastore-python/) 上了解更多关于该软件包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-datastore-python/blob/main/docs/document_loader.ipynb)

## 开始之前

要运行此笔记，您需要执行以下操作：

- [创建 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Datastore API](https://console.cloud.google.com/flows/enableapi?apiid=datastore.googleapis.com)

- [创建一个 Firestore 数据库模式中的 Datastore 数据库](https://cloud.google.com/datastore/docs/manage-databases)

在确认在笔记的运行环境中可以访问数据库之后，填写以下数值并在运行示例脚本之前运行单元格。

### 🦜🔗 库安装

集成位于其自己的 `langchain-google-datastore` 软件包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-datastore
```

**仅适用于 Colab**：取消注释以下单元格以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

- 运行 `gcloud config list`。

- 运行 `gcloud projects list`。

- 参阅支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填写您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 🔐 认证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 认证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记，使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### 保存文档

使用 `DatastoreSaver.upsert_documents(<documents>)` 保存 langchain 文档。默认情况下，它将尝试从文档元数据中的 `key` 提取实体键。

```python
from langchain_core.documents import Document
from langchain_google_datastore import DatastoreSaver
saver = DatastoreSaver()
data = [Document(page_content="Hello, World!")]
saver.upsert_documents(data)
```

#### 无键保存文档

如果指定了 `kind`，则将使用自动生成的 ID 存储文档。

```python
saver = DatastoreSaver("MyKind")
saver.upsert_documents(data)
```

### 通过 Kind 加载文档

使用 `DatastoreLoader.load()` 或 `DatastoreLoader.lazy_load()` 加载 langchain 文档。`lazy_load` 返回一个仅在迭代期间查询数据库的生成器。要初始化 `DatastoreLoader` 类，您需要提供：

1. `source` - 用于加载文档的源。它可以是查询的实例或要从中读取的 Datastore kind 的名称。

```python
from langchain_google_datastore import DatastoreLoader
loader = DatastoreLoader("MyKind")
data = loader.load()
```

### 通过查询加载文档

除了从 kind 加载文档外，我们还可以选择从查询加载文档。例如：

```python
from google.cloud import datastore
client = datastore.Client(database="non-default-db", namespace="custom_namespace")
query_load = client.query(kind="MyKind")
query_load.add_filter("region", "=", "west_coast")
loader_document = DatastoreLoader(query_load)
data = loader_document.load()
```

### 删除文档

使用 `DatastoreSaver.delete_documents(<documents>)` 从 Datastore 中删除一系列 langchain 文档。

```python
saver = DatastoreSaver()
saver.delete_documents(data)
keys_to_delete = [
    ["Kind1", "identifier"],
    ["Kind2", 123],
    ["Kind3", "identifier", "NestedKind", 456],
]
# 将忽略文档，并仅使用文档 ID。
saver.delete_documents(data, keys_to_delete)
```

## 高级用法

### 使用自定义文档页面内容和元数据加载文档

`page_content_properties` 和 `metadata_properties` 的参数将指定要写入 LangChain 文档 `page_content` 和 `metadata` 的实体属性。

```python
loader = DatastoreLoader(
    source="MyKind",
    page_content_fields=["data_field"],
    metadata_fields=["metadata_field"],
)
data = loader.load()
```

### 自定义页面内容格式

当 `page_content` 只包含一个字段时，信息将仅为该字段的值。否则，`page_content` 将采用 JSON 格式。

### 自定义连接和身份验证

```python
from google.auth import compute_engine
from google.cloud.firestore import Client
client = Client(database="non-default-db", creds=compute_engine.Credentials())
loader = DatastoreLoader(
    source="foo",
    client=client,
)
```