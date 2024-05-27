# Google BigQuery 向量搜索

[Google Cloud BigQuery 向量搜索](https://cloud.google.com/bigquery/docs/vector-search-intro) 允许您使用 GoogleSQL 进行语义搜索，使用向量索引快速获取近似结果，或者使用暴力搜索获取精确结果。

本教程演示了如何在 LangChain 中使用端到端数据和嵌入式管理系统，并在 BigQuery 中提供可扩展的语义搜索。

## 入门

### 安装库

```python
%pip install --upgrade --quiet  langchain langchain-google-vertexai google-cloud-bigquery
```

**仅适用于 Colab：** 取消下面的注释以重新启动内核，或者使用顶部的按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。

```python
# # 自动安装后重新启动内核，以便您的环境可以访问新的软件包
# import IPython
# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

## 开始之前

#### 设置您的项目 ID

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。

* 运行 `gcloud projects list`。

* 参阅支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @title 项目 { display-mode: "form" }
PROJECT_ID = ""  # @param {type:"string"}
# 设置项目 ID
! gcloud config set project {PROJECT_ID}
```

#### 设置区域

您还可以更改 BigQuery 使用的 `REGION` 变量。了解更多关于 [BigQuery 区域](https://cloud.google.com/bigquery/docs/locations#supported_locations) 的信息。

```python
# @title 区域 { display-mode: "form" }
REGION = "US"  # @param {type: "string"}
```

#### 设置数据集和表名称

它们将是您的 BigQuery 向量存储。

```python
# @title 数据集和表 { display-mode: "form" }
DATASET = "my_langchain_dataset"  # @param {type: "string"}
TABLE = "doc_and_vectors"  # @param {type: "string"}
```

### 对笔记本环境进行身份验证

- 如果您使用 **Colab** 运行此笔记本，请取消下面的注释并继续。

- 如果您使用 **Vertex AI Workbench**，请查看[此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。

```python
from google.colab import auth as google_auth
google_auth.authenticate_user()
```

## 演示：BigQueryVectorSearch

### 创建嵌入式类实例

您可能需要通过运行 `gcloud services enable aiplatform.googleapis.com --project {PROJECT_ID}`（将 `{PROJECT_ID}` 替换为您的项目名称）来在您的项目中启用 Vertex AI API。

您可以使用任何 [LangChain 嵌入式模型](/docs/integrations/text_embedding/)。

```python
from langchain_google_vertexai import VertexAIEmbeddings
embedding = VertexAIEmbeddings(
    model_name="textembedding-gecko@latest", project=PROJECT_ID
)
```

### 创建 BigQuery 数据集

如果数据集不存在，可以选择创建数据集。

```python
from google.cloud import bigquery
client = bigquery.Client(project=PROJECT_ID, location=REGION)
client.create_dataset(dataset=DATASET, exists_ok=True)
```

### 初始化 BigQueryVectorSearch 向量存储与现有的 BigQuery 数据集

```python
from langchain_community.vectorstores import BigQueryVectorSearch
from langchain_community.vectorstores.utils import DistanceStrategy
store = BigQueryVectorSearch(
    project_id=PROJECT_ID,
    dataset_name=DATASET,
    table_name=TABLE,
    location=REGION,
    embedding=embedding,
    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,
)
```

### 添加文本

```python
all_texts = ["Apples and oranges", "Cars and airplanes", "Pineapple", "Train", "Banana"]
metadatas = [{"len": len(t)} for t in all_texts]
store.add_texts(all_texts, metadatas=metadatas)
```

### 搜索文档

```python
query = "I'd like a fruit."
docs = store.similarity_search(query)
print(docs)
```

### 通过向量搜索文档

```python
query_vector = embedding.embed_query(query)
docs = store.similarity_search_by_vector(query_vector, k=2)
print(docs)
```

### 通过元数据筛选搜索文档

```python
# 这应该只返回 "Banana" 文档。
docs = store.similarity_search_by_vector(query_vector, filter={"len": 6})
print(docs)
```

### 使用 BigQuery 作业 ID 探索作业统计信息

```python
job_id = ""  # @param {type:"string"}
# 使用 BigQuery 作业 ID 调试和探索作业统计信息。
store.explore_job_stats(job_id)
```