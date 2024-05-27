# Vald

> [Vald](https://github.com/vdaas/vald) 是一个高度可扩展的分布式快速近似最近邻（ANN）密集向量搜索引擎。

本笔记展示了如何使用与 `Vald` 数据库相关的功能。

要运行本笔记，您需要一个正在运行的 Vald 集群。

有关更多信息，请查看[入门指南](https://github.com/vdaas/vald#get-started)。

请参阅[安装说明](https://github.com/vdaas/vald-client-python#install)。

```python
%pip install --upgrade --quiet  vald-client-python
```

## 基本示例

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Vald
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter
raw_documents = TextLoader("state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
documents = text_splitter.split_documents(raw_documents)
embeddings = HuggingFaceEmbeddings()
db = Vald.from_documents(documents, embeddings, host="localhost", port=8080)
```

```python
query = "总统对Ketanji Brown Jackson有何看法"
docs = db.similarity_search(query)
docs[0].page_content
```

### 通过向量进行相似性搜索

```python
embedding_vector = embeddings.embed_query(query)
docs = db.similarity_search_by_vector(embedding_vector)
docs[0].page_content
```

### 带分数的相似性搜索

```python
docs_and_scores = db.similarity_search_with_score(query)
docs_and_scores[0]
```

## 最大边际相关性搜索（MMR）

除了在 retriever 对象中使用相似性搜索外，还可以使用 `mmr` 作为检索器。

```python
retriever = db.as_retriever(search_type="mmr")
retriever.invoke(query)
```

或者直接使用 `max_marginal_relevance_search`：

```python
db.max_marginal_relevance_search(query, k=2, fetch_k=10)
```

## 使用安全连接的示例

为了运行本笔记，需要使用安全连接运行 Vald 集群。

以下是使用 [Athenz](https://github.com/AthenZ/athenz) 认证的以下配置的 Vald 集群示例。

入口（TLS） -> [授权代理](https://github.com/AthenZ/authorization-proxy)（检查 grpc 元数据中的 athenz-role-auth） -> vald-lb-gateway

```python
import grpc
with open("test_root_cacert.crt", "rb") as root:
    credentials = grpc.ssl_channel_credentials(root_certificates=root.read())
# 服务器使用需要刷新
with open(".ztoken", "rb") as ztoken:
    token = ztoken.read().strip()
metadata = [(b"athenz-role-auth", token)]
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Vald
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter
raw_documents = TextLoader("state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
documents = text_splitter.split_documents(raw_documents)
embeddings = HuggingFaceEmbeddings()
db = Vald.from_documents(
    documents,
    embeddings,
    host="localhost",
    port=443,
    grpc_use_secure=True,
    grpc_credentials=credentials,
    grpc_metadata=metadata,
)
```

```python
query = "总统对Ketanji Brown Jackson有何看法"
docs = db.similarity_search(query, grpc_metadata=metadata)
docs[0].page_content
```

### 通过向量进行相似性搜索

```python
embedding_vector = embeddings.embed_query(query)
docs = db.similarity_search_by_vector(embedding_vector, grpc_metadata=metadata)
docs[0].page_content
```

### 带分数的相似性搜索

```python
docs_and_scores = db.similarity_search_with_score(query, grpc_metadata=metadata)
docs_and_scores[0]
```

### 最大边际相关性搜索（MMR）

```python
retriever = db.as_retriever(
    search_kwargs={"search_type": "mmr", "grpc_metadata": metadata}
)
retriever.invoke(query, grpc_metadata=metadata)
```

或者：

```python
db.max_marginal_relevance_search(query, k=2, fetch_k=10, grpc_metadata=metadata)
```