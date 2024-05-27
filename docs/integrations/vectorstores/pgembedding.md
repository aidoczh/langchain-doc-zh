# Postgres 嵌入

[Postgres 嵌入](https://github.com/neondatabase/pg_embedding) 是一个开源的向量相似度搜索工具，用于 `Postgres` 数据库，它使用 `Hierarchical Navigable Small Worlds (HNSW)` 进行近似最近邻搜索。

它支持：

- 使用 HNSW 进行精确和近似最近邻搜索

- L2 距离

本笔记展示了如何使用 Postgres 向量数据库 (`PGEmbedding`)。

PGEmbedding 集成会为您创建 `pg_embedding` 扩展，但您需要运行以下 Postgres 查询来添加它：

```sql
CREATE EXTENSION embedding;
```

```python
# 安装必要的软件包
%pip install --upgrade --quiet langchain-openai
%pip install --upgrade --quiet psycopg2-binary
%pip install --upgrade --quiet tiktoken
```

将 OpenAI API 密钥添加到环境变量中以使用 `OpenAIEmbeddings`。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key:········
```

```python
## 加载环境变量
from typing import List, Tuple
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import PGEmbedding
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
os.environ["DATABASE_URL"] = getpass.getpass("Database Url:")
```

```output
Database Url:········
```

```python
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
connection_string = os.environ.get("DATABASE_URL")
collection_name = "state_of_the_union"
```

```python
db = PGEmbedding.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=collection_name,
    connection_string=connection_string,
)
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score: List[Tuple[Document, float]] = db.similarity_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```

## 在 Postgres 中使用向量存储

### 在 PG 中上传向量存储

```python
db = PGEmbedding.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=collection_name,
    connection_string=connection_string,
    pre_delete_collection=False,
)
```

### 创建 HNSW 索引

默认情况下，该扩展执行顺序扫描搜索，具有 100% 的召回率。您可以考虑创建 HNSW 索引以加速 `similarity_search_with_score` 的执行时间，实现近似最近邻 (ANN) 搜索。要在您的向量列上创建 HNSW 索引，使用 `create_hnsw_index` 函数：

```python
PGEmbedding.create_hnsw_index(
    max_elements=10000, dims=1536, m=8, ef_construction=16, ef_search=16
)
```

上面的函数等同于运行以下 SQL 查询：

```sql
CREATE INDEX ON vectors USING hnsw(vec) WITH (maxelements=10000, dims=1536, m=3, efconstruction=16, efsearch=16);
```

上述语句中使用的 HNSW 索引选项包括：

- maxelements：定义索引的最大元素数。这是一个必需参数。上面的示例值为 3。实际应用中会有一个更大的值，例如 1000000。一个 "元素" 指的是数据集中的数据点（向量），它在 HNSW 图中表示为一个节点。通常，您会将此选项设置为能够容纳数据集中行数的值。

- dims：定义向量数据中的维数。这是一个必需参数。上面的示例中使用了一个小值。如果您正在存储使用 OpenAI 的 text-embedding-ada-002 模型生成的数据，该模型支持 1536 维，您可以定义一个值为 1536 的维数。

- m：定义在图构建过程中为每个节点创建的双向链接（也称为 "边"）的最大数量。

支持以下额外的索引选项：

- efConstruction：定义索引构建过程中考虑的最近邻数。默认值为 32。

- efsearch：定义索引搜索过程中考虑的最近邻数。默认值为 32。

有关如何配置这些选项以影响 HNSW 算法的信息，请参阅 [调整 HNSW 算法](https://neon.tech/docs/extensions/pg_embedding#tuning-the-hnsw-algorithm)。

### 从 PG 中检索向量存储

```python
store = PGEmbedding(
    connection_string=connection_string,
    embedding_function=embeddings,
    collection_name=collection_name,
)
retriever = store.as_retriever()
```

```python
retriever
```

```VectorStoreRetriever(vectorstore=<langchain_community.vectorstores.pghnsw.HNSWVectoreStore object at 0x121d3c8b0>, search_type='similarity', search_kwargs={})
```

```python
db1 = PGEmbedding.from_existing_index(
    embedding=embeddings,
    collection_name=collection_name,
    pre_delete_collection=False,
    connection_string=connection_string,
)
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score: List[Tuple[Document, float]] = db1.similarity_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```

```