# Google Cloud SQL for PostgreSQL

> [Cloud SQL](https://cloud.google.com/sql) 是一种完全托管的关系型数据库服务，提供高性能、无缝集成和可扩展性强的特点。它支持 PostgreSQL、MySQL 和 SQL Server 数据库引擎。通过使用 Cloud SQL 的 Langchain 集成，您可以扩展数据库应用程序，构建基于人工智能的体验。

本文介绍如何使用 `Cloud SQL for PostgreSQL` 将向量嵌入存储到 `PostgresVectorStore` 类中。

在 [GitHub](https://github.com/googleapis/langchain-google-cloud-sql-pg-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-cloud-sql-pg-python/blob/main/docs/vector_store.ipynb)

## 开始之前

在运行这个 notebook 之前，您需要完成以下步骤：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)

* [启用 Cloud SQL Admin API](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin.googleapis.com)

* [创建一个 Cloud SQL 实例](https://cloud.google.com/sql/docs/postgres/connect-instance-auth-proxy#create-instance)

* [创建一个 Cloud SQL 数据库](https://cloud.google.com/sql/docs/postgres/create-manage-databases)

* [向数据库添加用户](https://cloud.google.com/sql/docs/postgres/create-manage-users)

### 🦜🔗 安装库

安装集成库 `langchain-google-cloud-sql-pg` 和嵌入式服务库 `langchain-google-vertexai`。

```python
%pip install --upgrade --quiet  langchain-google-cloud-sql-pg langchain-google-vertexai
```

**仅适用于 Colab：**取消注释以下单元格以重新启动内核，或使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 在安装后自动重新启动内核，以便您的环境可以访问新的包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 身份验证

以 IAM 用户的身份验证到 Google Cloud，以便在此 notebook 中访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此 notebook，请使用下面的单元格并继续。

* 如果您使用 Vertex AI Workbench，请查看[这里的设置说明](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)。

```python
from google.colab import auth
auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目

设置您的 Google Cloud 项目，以便您可以在此 notebook 中使用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面的值中填入您的 Google Cloud 项目 ID，然后运行该单元格。
PROJECT_ID = "my-project-id"  # @param {type:"string"}
# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

## 基本用法

### 设置 Cloud SQL 数据库值

在 [Cloud SQL 实例页面](https://console.cloud.google.com/sql?_ga=2.223735448.2062268965.1707700487-2088871159.1707257687)中找到您的数据库值。

```python
# @title 在此处设置您的值 { display-mode: "form" }
REGION = "us-central1"  # @param {type: "string"}
INSTANCE = "my-pg-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "vector_store"  # @param {type: "string"}
```

### PostgresEngine 连接池

将 Cloud SQL 作为向量存储的要求之一是一个 `PostgresEngine` 对象。`PostgresEngine` 配置了一个连接池到您的 Cloud SQL 数据库，使您的应用程序能够成功连接，并遵循行业最佳实践。

要使用 `PostgresEngine.from_instance()` 创建一个 `PostgresEngine`，您只需要提供以下 4 个参数：

1. `project_id`：Cloud SQL 实例所在的 Google Cloud 项目的项目 ID。

2. `region`：Cloud SQL 实例所在的区域。

3. `instance`：Cloud SQL 实例的名称。

4. `database`：要连接到的 Cloud SQL 实例上的数据库的名称。

默认情况下，将使用 [IAM 数据库身份验证](https://cloud.google.com/sql/docs/postgres/iam-authentication#iam-db-auth) 作为数据库身份验证的方法。该库使用来自环境的 [应用程序默认凭据 (ADC)](https://cloud.google.com/docs/authentication/application-default-credentials) 的 IAM 主体。

有关 IAM 数据库身份验证的更多信息，请参见：

* [为 IAM 数据库认证配置实例](https://cloud.google.com/sql/docs/postgres/create-edit-iam-instances)

* [使用 IAM 数据库认证管理用户](https://cloud.google.com/sql/docs/postgres/add-manage-iam-users)

还可以选择使用用户名和密码访问 Cloud SQL 数据库的[内置数据库认证](https://cloud.google.com/sql/docs/postgres/built-in-authentication)。只需为 `PostgresEngine.from_instance()` 提供可选的 `user` 和 `password` 参数：

* `user`：用于内置数据库认证和登录的数据库用户

* `password`：用于内置数据库认证和登录的数据库密码

**注意**：本教程演示了异步接口。所有异步方法都有对应的同步方法。

```python
from langchain_google_cloud_sql_pg import PostgresEngine
engine = await PostgresEngine.afrom_instance(
    project_id=PROJECT_ID, region=REGION, instance=INSTANCE, database=DATABASE
)
```

### 初始化表

`PostgresVectorStore` 类需要一个数据库表。`PostgresEngine` 引擎有一个辅助方法 `init_vectorstore_table()`，可用于为您创建具有适当模式的表。

```python
from langchain_google_cloud_sql_pg import PostgresEngine
await engine.ainit_vectorstore_table(
    table_name=TABLE_NAME,
    vector_size=768,  # VertexAI 模型（textembedding-gecko@latest）的向量大小
)
```

### 创建嵌入类实例

您可以使用任何[LangChain 嵌入模型](/docs/integrations/text_embedding/)。您可能需要启用 Vertex AI API 以使用 `VertexAIEmbeddings`。我们建议为生产环境设置嵌入模型的版本，了解更多关于[文本嵌入模型](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/text-embeddings)的信息。

```python
# 启用 Vertex AI API
!gcloud services enable aiplatform.googleapis.com
```

```python
from langchain_google_vertexai import VertexAIEmbeddings
embedding = VertexAIEmbeddings(
    model_name="textembedding-gecko@latest", project=PROJECT_ID
)
```

### 初始化默认的 PostgresVectorStore

```python
from langchain_google_cloud_sql_pg import PostgresVectorStore
store = await PostgresVectorStore.create(  # 使用 .create() 初始化异步向量存储
    engine=engine,
    table_name=TABLE_NAME,
    embedding_service=embedding,
)
```

### 添加文本

```python
import uuid
all_texts = ["Apples and oranges", "Cars and airplanes", "Pineapple", "Train", "Banana"]
metadatas = [{"len": len(t)} for t in all_texts]
ids = [str(uuid.uuid4()) for _ in all_texts]
await store.aadd_texts(all_texts, metadatas=metadatas, ids=ids)
```

### 删除文本

```python
await store.adelete([ids[1]])
```

### 搜索文档

```python
query = "I'd like a fruit."
docs = await store.asimilarity_search(query)
print(docs)
```

### 按向量搜索文档

```python
query_vector = embedding.embed_query(query)
docs = await store.asimilarity_search_by_vector(query_vector, k=2)
print(docs)
```

## 添加索引

通过应用向量索引加快向量搜索查询速度。了解更多关于[向量索引](https://cloud.google.com/blog/products/databases/faster-similarity-search-performance-with-pgvector-indexes)的信息。

```python
from langchain_google_cloud_sql_pg.indexes import IVFFlatIndex
index = IVFFlatIndex()
await store.aapply_vector_index(index)
```

### 重新索引

```python
await store.areindex()  # 使用默认索引名称重新索引
```

### 删除索引

```python
await store.aadrop_vector_index()  # 使用默认名称删除索引
```

## 创建自定义向量存储

向量存储可以利用关系数据来过滤相似性搜索。

创建具有自定义元数据列的表。

```python
from langchain_google_cloud_sql_pg import Column
# 设置表名
TABLE_NAME = "vectorstore_custom"
await engine.ainit_vectorstore_table(
    table_name=TABLE_NAME,
    vector_size=768,  # VertexAI 模型：textembedding-gecko@latest
    metadata_columns=[Column("len", "INTEGER")],
)
# 初始化 PostgresVectorStore
custom_store = await PostgresVectorStore.create(
    engine=engine,
    table_name=TABLE_NAME,
    embedding_service=embedding,
    metadata_columns=["len"],
    # 通过自定义表模式连接到现有的 VectorStore：
    # id_column="uuid",
    # content_column="documents",
    # embedding_column="vectors",
)
```

### 使用元数据过滤搜索文档

```python
import uuid
# 向 Vector Store 添加文本
all_texts = ["Apples and oranges", "Cars and airplanes", "Pineapple", "Train", "Banana"]
metadatas = [{"len": len(t)} for t in all_texts]
ids = [str(uuid.uuid4()) for _ in all_texts]
await store.aadd_texts(all_texts, metadatas=metadatas, ids=ids)
# 在搜索中使用过滤器
docs = await custom_store.asimilarity_search_by_vector(query_vector, filter="len >= 6")
print(docs)
```

抱歉，你似乎忘记输入要翻译的英文内容了。请提供英文段落，我会尽快帮你翻译成中文科普风格的文章。