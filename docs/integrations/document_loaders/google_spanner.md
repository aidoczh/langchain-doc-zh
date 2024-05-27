# 谷歌Spanner

[Spanner](https://cloud.google.com/spanner) 是一个高度可扩展的数据库，它将无限可扩展性与关系语义结合在一起，例如次要索引、强一致性、模式和 SQL，提供了一个简单的解决方案，可实现 99.999% 的可用性。

这个笔记本介绍了如何使用 [Spanner](https://cloud.google.com/spanner) 来使用 `SpannerLoader` 和 `SpannerDocumentSaver` 来 [保存、加载和删除 langchain 文档](/docs/how_to#document-loaders)。

在 [GitHub](https://github.com/googleapis/langchain-google-spanner-python/) 上了解更多关于这个包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-spanner-python/blob/main/docs/document_loader.ipynb)

## 开始之前

要运行这个笔记本，您需要完成以下步骤：

- [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

- [启用 Cloud Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)

- [创建一个 Spanner 实例](https://cloud.google.com/spanner/docs/create-manage-instances)

- [创建一个 Spanner 数据库](https://cloud.google.com/spanner/docs/create-manage-databases)

- [创建一个 Spanner 表](https://cloud.google.com/spanner/docs/create-query-database-console#create-schema)

在确认在笔记本的运行环境中可以访问数据库之后，填写以下数值并在运行示例脚本之前运行单元格。

```python
# @markdown 请指定一个实例 ID、一个数据库和一个表以进行演示。
INSTANCE_ID = "test_instance"  # @param {type:"string"}
DATABASE_ID = "test_database"  # @param {type:"string"}
TABLE_NAME = "test_table"  # @param {type:"string"}
```

### 🦜🔗 库安装

集成在自己的 `langchain-google-spanner` 包中，所以我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-spanner langchain
```

**仅限 Colab**：取消下面的单元格的注释以重新启动内核，或者使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动在安装后重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
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

### 🔐 认证

作为在这个笔记本中登录的 IAM 用户，对 Google Cloud 进行身份验证，以便访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行这个笔记本，请使用下面的单元格并继续。

- 如果您使用 Vertex AI Workbench，请查看[这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

## 基本用法

### 保存文档

使用 `SpannerDocumentSaver.add_documents(<documents>)` 保存 langchain 文档。要初始化 `SpannerDocumentSaver` 类，您需要提供 3 个东西：

1. `instance_id` - 从中加载数据的 Spanner 实例。

2. `database_id` - 从中加载数据的 Spanner 数据库实例。

3. `table_name` - Spanner 数据库中存储 langchain 文档的表的名称。

```python
from langchain_core.documents import Document
from langchain_google_spanner import SpannerDocumentSaver
test_docs = [
    Document(
        page_content="Apple Granny Smith 150 0.99 1",
        metadata={"fruit_id": 1},
    ),
    Document(
        page_content="Banana Cavendish 200 0.59 0",
        metadata={"fruit_id": 2},
    ),
    Document(
        page_content="Orange Navel 80 1.29 1",
        metadata={"fruit_id": 3},
    ),
]
saver = SpannerDocumentSaver(
    instance_id=INSTANCE_ID,
    database_id=DATABASE_ID,
    table_name=TABLE_NAME,
)
saver.add_documents(test_docs)
```

### 从 Spanner 查询文档

有关连接到 Spanner 表的更多详细信息，请查看[Python SDK 文档](https://cloud.google.com/python/docs/reference/spanner/latest)。

#### 从表中加载文档

使用 `SpannerLoader.load()` 或 `SpannerLoader.lazy_load()` 加载 langchain 文档。`lazy_load` 返回一个仅在迭代期间查询数据库的生成器。要初始化 `SpannerLoader` 类，您需要提供以下信息：

1. `instance_id` - 从中加载数据的 Spanner 实例。

2. `database_id` - 从中加载数据的 Spanner 数据库实例。

3. `query` - 数据库方言的查询。

```python
from langchain_google_spanner import SpannerLoader
query = f"SELECT * from {TABLE_NAME}"
loader = SpannerLoader(
    instance_id=INSTANCE_ID,
    database_id=DATABASE_ID,
    query=query,
)
for doc in loader.lazy_load():
    print(doc)
    break
```

### 删除文档

使用 `SpannerDocumentSaver.delete(<documents>)` 从表中删除 langchain 文档的列表。

```python
docs = loader.load()
print("删除前的文档:", docs)
doc = test_docs[0]
saver.delete([doc])
print("删除后的文档:", loader.load())
```

## 高级用法

### 自定义客户端

默认情况下创建的客户端是默认客户端。要显式传递 `credentials` 和 `project`，可以将自定义客户端传递给构造函数。

```python
from google.cloud import spanner
from google.oauth2 import service_account
creds = service_account.Credentials.from_service_account_file("/path/to/key.json")
custom_client = spanner.Client(project="my-project", credentials=creds)
loader = SpannerLoader(
    INSTANCE_ID,
    DATABASE_ID,
    query,
    client=custom_client,
)
```

### 自定义文档页面内容和元数据

加载器将返回具有特定数据列的文档列表作为页面内容。所有其他数据列将被添加到元数据中。每行都成为一个文档。

#### 自定义页面内容格式

SpannerLoader 假定存在一个名为 `page_content` 的列。可以像这样更改默认设置：

```python
custom_content_loader = SpannerLoader(
    INSTANCE_ID, DATABASE_ID, query, content_columns=["custom_content"]
)
```

如果指定了多个列，则页面内容的字符串格式默认为 `text`（空格分隔的字符串连接）。用户可以指定其他格式，包括 `text`、`JSON`、`YAML`、`CSV`。

#### 自定义元数据格式

SpannerLoader 假定存在一个名为 `langchain_metadata` 的元数据列，其中存储了 JSON 数据。元数据列将被用作基本字典。默认情况下，所有其他列数据将被添加并可能覆盖原始值。可以像这样更改默认设置：

```python
custom_metadata_loader = SpannerLoader(
    INSTANCE_ID, DATABASE_ID, query, metadata_columns=["column1", "column2"]
)
```

#### 自定义 JSON 元数据列名称

默认情况下，加载器使用 `langchain_metadata` 作为基本字典。可以自定义选择要用作文档元数据基本字典的 JSON 列。

```python
custom_metadata_json_loader = SpannerLoader(
    INSTANCE_ID, DATABASE_ID, query, metadata_json_column="another-json-column"
)
```

### 自定义陈旧度

默认的[陈旧度](https://cloud.google.com/python/docs/reference/spanner/latest/snapshot-usage#beginning-a-snapshot)为 15 秒。可以通过指定较弱的限制（可以是根据给定时间戳执行所有读取的操作），或者是过去一定时间段内的操作来自定义。

```python
import datetime
timestamp = datetime.datetime.utcnow()
custom_timestamp_loader = SpannerLoader(
    INSTANCE_ID,
    DATABASE_ID,
    query,
    staleness=timestamp,
)
```

```python
duration = 20.0
custom_duration_loader = SpannerLoader(
    INSTANCE_ID,
    DATABASE_ID,
    query,
    staleness=duration,
)
```

### 打开数据增强

默认情况下，加载器不会使用 [数据增强](https://cloud.google.com/spanner/docs/databoost/databoost-overview)，因为它会带来额外的成本，并且需要额外的 IAM 权限。但是，用户可以选择打开它。

```python
custom_databoost_loader = SpannerLoader(
    INSTANCE_ID,
    DATABASE_ID,
    query,
    databoost=True,
)
```

### 自定义客户端

默认情况下创建的客户端是默认客户端。要显式传递 `credentials` 和 `project`，可以将自定义客户端传递给构造函数。

```python
from google.cloud import spanner
custom_client = spanner.Client(project="my-project", credentials=creds)
saver = SpannerDocumentSaver(
    INSTANCE_ID,
    DATABASE_ID,
    TABLE_NAME,
    client=custom_client,
)
```

### SpannerDocumentSaver 的自定义初始化

SpannerDocumentSaver 允许自定义初始化。这允许用户指定如何将文档保存到表中。

content_column: 这将用作文档页面内容的列名。默认为 `page_content`。

```python
metadata_columns: 如果文档的元数据中存在该键，则将这些元数据保存到特定的列中。
metadata_json_column: 这将是特殊 JSON 列的列名，默认为 `langchain_metadata`。
custom_saver = SpannerDocumentSaver(
    INSTANCE_ID,
    DATABASE_ID,
    TABLE_NAME,
    content_column="my-content",
    metadata_columns=["foo"],
    metadata_json_column="my-special-json-column",
)
```

### 初始化 Spanner 的自定义模式

SpannerDocumentSaver 将具有 `init_document_table` 方法，用于创建一个新表来存储具有自定义模式的文档。

```python
from langchain_google_spanner import Column
new_table_name = "my_new_table"
SpannerDocumentSaver.init_document_table(
    INSTANCE_ID,
    DATABASE_ID,
    new_table_name,
    content_column="my-page-content",
    metadata_columns=[
        Column("category", "STRING(36)", True),
        Column("price", "FLOAT64", False),
    ],
)
```