# DashVector

> [DashVector](https://help.aliyun.com/document_detail/2510225.html) 是一项完全托管的 vectorDB 服务，支持高维稠密和稀疏向量、实时插入和过滤搜索。它被设计为能够自动扩展，并能够适应不同的应用需求。

本笔记展示了如何使用与 `DashVector` 向量数据库相关的功能。

要使用 DashVector，您必须拥有一个 API 密钥。

以下是[安装说明](https://help.aliyun.com/document_detail/2510223.html)。

## 安装

```python
%pip install --upgrade --quiet  dashvector dashscope
```

我们想要使用 `DashScopeEmbeddings`，因此我们还必须获取 Dashscope API 密钥。

```python
import getpass
import os
os.environ["DASHVECTOR_API_KEY"] = getpass.getpass("DashVector API Key:")
os.environ["DASHSCOPE_API_KEY"] = getpass.getpass("DashScope API Key:")
```

## 示例

```python
from langchain_community.embeddings.dashscope import DashScopeEmbeddings
from langchain_community.vectorstores import DashVector
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = DashScopeEmbeddings()
```

我们可以从文档中创建 DashVector。

```python
dashvector = DashVector.from_documents(docs, embeddings)
query = "What did the president say about Ketanji Brown Jackson"
docs = dashvector.similarity_search(query)
print(docs)
```

我们可以添加带有元数据和 ID 的文本，并使用元数据过滤进行搜索。

```python
texts = ["foo", "bar", "baz"]
metadatas = [{"key": i} for i in range(len(texts))]
ids = ["0", "1", "2"]
dashvector.add_texts(texts, metadatas=metadatas, ids=ids)
docs = dashvector.similarity_search("foo", filter="key = 2")
print(docs)
```

```output
[Document(page_content='baz', metadata={'key': 2})]
```

### 操作带 `partition` 参数的频段

`partition` 参数默认为默认值，如果传入不存在的 `partition` 参数，则 `partition` 将被自动创建。

```python
texts = ["foo", "bar", "baz"]
metadatas = [{"key": i} for i in range(len(texts))]
ids = ["0", "1", "2"]
partition = "langchain"
# 添加文本
dashvector.add_texts(texts, metadatas=metadatas, ids=ids, partition=partition)
# 相似性搜索
query = "What did the president say about Ketanji Brown Jackson"
docs = dashvector.similarity_search(query, partition=partition)
# 删除
dashvector.delete(ids=ids, partition=partition)
```