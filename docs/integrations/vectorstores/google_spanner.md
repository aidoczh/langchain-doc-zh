# 谷歌 Spanner

> [Spanner](https://cloud.google.com/spanner) 是一种高度可扩展的数据库，它将无限可扩展性与关系语义（如次要索引、强一致性、模式和 SQL）结合在一个简单的解决方案中，提供 99.999% 的可用性。

本笔记本介绍了如何使用 `Spanner` 进行向量搜索，使用 `SpannerVectorStore` 类。

在 [GitHub](https://github.com/googleapis/langchain-google-spanner-python/) 上了解有关该软件包的更多信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-spanner-python/blob/main/docs/vector_store.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

* [创建 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

* [启用 Cloud Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)

* [创建 Spanner 实例](https://cloud.google.com/spanner/docs/create-manage-instances)

* [创建 Spanner 数据库](https://cloud.google.com/spanner/docs/create-manage-databases)

### 🦜🔗 安装库

集成位于其自己的 `langchain-google-spanner` 软件包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-spanner
```

```output
Note: you may need to restart the kernel to use updated packages.
```

**仅限 Colab：** 取消注释以下单元格以重新启动内核，或使用按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 安装后自动重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 认证

作为在此笔记本中登录的 IAM 用户，进行 Google Cloud 认证，以便访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。

* 如果您使用 Vertex AI Workbench，请查看[此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 参见支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填入您的 Google Cloud 项目 ID，然后运行单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 启用 API

`langchain-google-spanner` 软件包要求您在 Google Cloud 项目中[启用 Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)。

```python
# 启用 Spanner API
!gcloud services enable spanner.googleapis.com
```

## 基本用法

### 设置 Spanner 数据库值

在 [Spanner 实例页面](https://console.cloud.google.com/spanner?_ga=2.223735448.2062268965.1707700487-2088871159.1707257687)中找到您的数据库值。

```python
# @title 在此处设置您的值 { display-mode: "form" }
INSTANCE = "my-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "vectors_search_data"  # @param {type: "string"}
```

### 初始化表

`SpannerVectorStore` 类实例需要具有 id、content 和 embeddings 列的数据库表。

可以使用辅助方法 `init_vector_store_table()` 来为您创建具有适当模式的表。

```python
from langchain_google_spanner import SecondaryIndex, SpannerVectorStore, TableColumn
SpannerVectorStore.init_vector_store_table(
    instance_id=INSTANCE,
    database_id=DATABASE,
    table_name=TABLE_NAME,
    id_column="row_id",
    metadata_columns=[
        TableColumn(name="metadata", type="JSON", is_null=True),
        TableColumn(name="title", type="STRING(MAX)", is_null=False),
    ],
    secondary_indexes=[
        SecondaryIndex(index_name="row_id_and_title", columns=["row_id", "title"])
    ],
)
```

### 创建嵌入类实例

您可以使用任何[LangChain 嵌入模型](/docs/integrations/text_embedding/)。

您可能需要启用 Vertex AI API 以使用 `VertexAIEmbeddings`。我们建议为生产设置嵌入模型的版本，了解更多关于[文本嵌入模型](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/text-embeddings)的信息。

```python
# 启用 Vertex AI API
!gcloud services enable aiplatform.googleapis.com
```

```python
from langchain_google_vertexai import VertexAIEmbeddings
embeddings = VertexAIEmbeddings(
    model_name="textembedding-gecko@latest", project=PROJECT_ID
)
### SpannerVectorStore
要初始化 `SpannerVectorStore` 类，您需要提供 4 个必需参数，其他参数是可选的，只有在与默认参数不同时才需要传递。
1. `instance_id` - Spanner 实例的名称
2. `database_id` - Spanner 数据库的名称
3. `table_name` - 数据库中用于存储文档及其嵌入的表的名称
4. `embedding_service` - 用于生成嵌入的嵌入实现
```python

db = SpannerVectorStore(

    instance_id=INSTANCE,

    database_id=DATABASE,

    table_name=TABLE_NAME,

    ignore_metadata_columns=[],

    embedding_service=embeddings,

    metadata_json_column="metadata",

)

```
#### 🔐 添加文档
向向量存储中添加文档。
```python
import uuid
from langchain_community.document_loaders import HNLoader
loader = HNLoader("https://news.ycombinator.com/item?id=34817881")
documents = loader.load()
ids = [str(uuid.uuid4()) for _ in range(len(documents))]
```
#### 🔐 搜索文档
使用相似性搜索在向量存储中搜索文档。
```python
db.similarity_search(query="Explain me vector store?", k=3)
```
#### 🔐 搜索文档
使用最大边际相关性搜索在向量存储中搜索文档。
```python
db.max_marginal_relevance_search("Testing the langchain integration with spanner", k=3)
```
#### 🔐 删除文档
要从向量存储中删除文档，使用初始化 VectorStore 时对应于 `row_id` 列中的值的 ID。
```python
db.delete(ids=["id1", "id2"])
```
#### 🔐 删除文档
要从向量存储中删除文档，您可以利用文档本身。在 VectorStore 初始化期间提供的内容列和元数据列将用于查找与文档对应的行。然后将删除任何匹配的行。
```python
db.delete(documents=[documents[0], documents[1]])
```