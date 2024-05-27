# Tair

[Tair](https://www.alibabacloud.com/help/en/tair/latest/what-is-tair) 是由阿里云开发的云原生内存数据库服务。它提供丰富的数据模型和企业级能力，以支持您的实时在线场景，同时与开源的 `Redis` 完全兼容。`Tair` 还引入了基于新的非易失性内存（NVM）存储介质的持久内存优化实例。

这个笔记本展示了如何使用与 `Tair` 向量数据库相关的功能。

要运行，您应该有一个正在运行的 `Tair` 实例。

```python
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import Tair
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = FakeEmbeddings(size=128)
```

使用 `TAIR_URL` 环境变量连接到 Tair

```
export TAIR_URL="redis://{username}:{password}@{tair_address}:{tair_port}"
```

或者使用关键字参数 `tair_url`。

然后将文档和嵌入存储到 Tair。

```python
tair_url = "redis://localhost:6379"
# 如果索引已经存在，则先删除
Tair.drop_index(tair_url=tair_url)
vector_store = Tair.from_documents(docs, embeddings, tair_url=tair_url)
```

查询相似文档。

```python
query = "总统对Ketanji Brown Jackson有何看法"
docs = vector_store.similarity_search(query)
docs[0]
```

Tair 混合搜索索引构建

```python
# 如果索引已经存在，则先删除
Tair.drop_index(tair_url=tair_url)
vector_store = Tair.from_documents(
    docs, embeddings, tair_url=tair_url, index_params={"lexical_algorithm": "bm25"}
)
```

Tair 混合搜索

```python
query = "总统对Ketanji Brown Jackson有何看法"
# hybrid_ratio: 0.5 混合搜索, 0.9999 向量搜索, 0.0001 文本搜索
kwargs = {"TEXT": query, "hybrid_ratio": 0.5}
docs = vector_store.similarity_search(query, **kwargs)
docs[0]
```