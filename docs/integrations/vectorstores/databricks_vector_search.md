# Databricks Vector Search

Databricks Vector Search 是一个无服务器的相似度搜索引擎，允许您在向量数据库中存储数据的向量表示，包括元数据。使用 Vector Search，您可以从由 Unity Catalog 管理的 Delta 表中创建自动更新的向量搜索索引，并使用简单的 API 进行查询以返回最相似的向量。

本笔记本展示了如何在 Databricks Vector Search 中使用 LangChain。

安装 `databricks-vectorsearch` 和本笔记本中使用的相关 Python 包。

```python
%pip install --upgrade --quiet  langchain-core databricks-vectorsearch langchain-openai tiktoken
```

使用 `OpenAIEmbeddings` 进行嵌入。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

拆分文档并获取嵌入。

```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
emb_dim = len(embeddings.embed_query("hello"))
```

## 设置 Databricks Vector Search 客户端

```python
from databricks.vector_search.client import VectorSearchClient
vsc = VectorSearchClient()
```

## 创建 Vector Search 端点

此端点用于创建和访问向量搜索索引。

```python
vsc.create_endpoint(name="vector_search_demo_endpoint", endpoint_type="STANDARD")
```

## 创建直接向量访问索引

直接向量访问索引支持通过 REST API 或 SDK 直接读取和写入嵌入向量和元数据。对于此索引，您自己管理嵌入向量和索引更新。

```python
vector_search_endpoint_name = "vector_search_demo_endpoint"
index_name = "ml.llm.demo_index"
index = vsc.create_direct_access_index(
    endpoint_name=vector_search_endpoint_name,
    index_name=index_name,
    primary_key="id",
    embedding_dimension=emb_dim,
    embedding_vector_column="text_vector",
    schema={
        "id": "string",
        "text": "string",
        "text_vector": "array<float>",
        "source": "string",
    },
)
index.describe()
```

```python
from langchain_community.vectorstores import DatabricksVectorSearch
dvs = DatabricksVectorSearch(
    index, text_column="text", embedding=embeddings, columns=["source"]
)
```

## 将文档添加到索引中

```python
dvs.add_documents(docs)
```

## 相似度搜索

```python
query = "What did the president say about Ketanji Brown Jackson"
dvs.similarity_search(query)
print(docs[0].page_content)
```

## 使用 Delta Sync Index

您还可以使用 `DatabricksVectorSearch` 在 Delta Sync Index 中进行搜索。Delta Sync Index 会自动从 Delta 表同步。您不需要手动调用 `add_text`/`add_documents`。有关更多详细信息，请参阅 [Databricks 文档页面](https://docs.databricks.com/en/generative-ai/vector-search.html#delta-sync-index-with-managed-embeddings)。

```python
dvs_delta_sync = DatabricksVectorSearch("catalog_name.schema_name.delta_sync_index")
dvs_delta_sync.similarity_search(query)
```