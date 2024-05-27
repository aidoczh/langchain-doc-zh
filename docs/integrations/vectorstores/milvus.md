# Milvus

[Milvus](https://milvus.io/docs/overview.md) 是一个数据库，用于存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大规模嵌入向量。

这篇笔记展示了如何使用与 Milvus 向量数据库相关的功能。

要运行，您应该已经启动并运行了一个[Milvus 实例](https://milvus.io/docs/install_standalone-docker.md)。

```python
%pip install --upgrade --quiet  pymilvus
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key:········
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Milvus
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"host": "127.0.0.1", "port": "19530"},
)
```

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
```

```python
docs[0].page_content
```

```output
'Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.'
```

### 使用 Milvus 集合对数据进行分隔

您可以将不同的不相关文档存储在同一个 Milvus 实例中的不同集合中，以保持上下文。

以下是如何创建一个新集合的方法：

```python
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    collection_name="collection_1",
    connection_args={"host": "127.0.0.1", "port": "19530"},
)
```

这是如何检索存储的集合：

```python
vector_db = Milvus(
    embeddings,
    connection_args={"host": "127.0.0.1", "port": "19530"},
    collection_name="collection_1",
)
```

检索后，您可以像往常一样进行查询。

### 按用户检索

在构建检索应用程序时，通常需要考虑多个用户。这意味着您可能不仅为一个用户存储数据，还为许多不同用户存储数据，并且它们不应该能够看到彼此的数据。

Milvus 建议使用 [partition_key](https://milvus.io/docs/multi_tenancy.md#Partition-key-based-multi-tenancy) 来实现多租户，以下是一个示例。

```python
from langchain_core.documents import Document
docs = [
    Document(page_content="i worked at kensho", metadata={"namespace": "harrison"}),
    Document(page_content="i worked at facebook", metadata={"namespace": "ankush"}),
]
vectorstore = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"host": "127.0.0.1", "port": "19530"},
    drop_old=True,
    partition_key_field="namespace",  # 使用 "namespace" 字段作为分区键
)
```

要使用分区键进行搜索，您应该在搜索请求的布尔表达式中包含以下内容之一：

`search_kwargs={"expr": '<partition_key> == "xxxx"}`

`search_kwargs={"expr": '<partition_key> == in ["xxx", "xxx"]}`

请用指定的分区键名称替换 `<partition_key>`。

Milvus 根据指定的分区键更改分区，根据分区键过滤实体，并在过滤后的实体中进行搜索。

```python
# 这将仅获取 Ankush 的文档
vectorstore.as_retriever(search_kwargs={"expr": 'namespace == "ankush"'}).invoke(
    "where did i work?"
)
```

```output
[Document(page_content='i worked at facebook', metadata={'namespace': 'ankush'})]
```

```python
# 这将仅获取 Harrison 的文档
vectorstore.as_retriever(search_kwargs={"expr": 'namespace == "harrison"'}).invoke(
    "where did i work?"
)
```

```output
[文档（页面内容='我在肯肖工作过'，元数据={'命名空间': '哈里森'}）]
```

**要删除或更新（插入/更新）一个或多个实体：**

```python
from langchain_core.documents import Document
# 插入数据样本
docs = [
    Document(page_content="foo", metadata={"id": 1}),
    Document(page_content="bar", metadata={"id": 2}),
    Document(page_content="baz", metadata={"id": 3}),
]
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"host": "127.0.0.1", "port": "19530"},
)
# 使用表达式搜索主键（pks）
expr = "id in [1,2]"
pks = vector_db.get_pks(expr)
# 按主键（pks）删除实体
result = vector_db.delete(pks)
# 更新/插入
new_docs = [
    Document(page_content="new_foo", metadata={"id": 1}),
    Document(page_content="new_bar", metadata={"id": 2}),
    Document(page_content="upserted_bak", metadata={"id": 3}),
]
upserted_pks = vector_db.upsert(pks, new_docs)
```