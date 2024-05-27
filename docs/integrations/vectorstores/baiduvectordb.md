# 百度 VectorDB

>[百度 VectorDB](https://cloud.baidu.com/product/vdb.html) 是由百度智能云精心开发和完全管理的强大的企业级分布式数据库服务。它以其出色的存储、检索和分析多维向量数据的能力而脱颖而出。在其核心，VectorDB 使用百度专有的 "Mochow" 向量数据库内核，确保高性能、可用性和安全性，以及卓越的可扩展性和用户友好性。

该数据库服务支持多种索引类型和相似度计算方法，以满足各种用例。VectorDB 的一个突出特点是其能够管理高达100亿的巨大向量规模，同时保持令人瞩目的查询性能，支持每秒百万级的查询量，并具有毫秒级的查询延迟。

本笔记本展示了如何使用与百度 VectorDB 相关的功能。

要运行，您应该拥有一个[数据库实例](https://cloud.baidu.com/doc/VDB/s/hlrsoazuf)。

```python
!pip3 install pymochow
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import BaiduVectorDB
from langchain_community.vectorstores.baiduvectordb import ConnectionParams
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = FakeEmbeddings(size=128)
```

```python
conn_params = ConnectionParams(
    endpoint="http://192.168.xx.xx:xxxx", account="root", api_key="****"
)
vector_db = BaiduVectorDB.from_documents(
    docs, embeddings, connection_params=conn_params, drop_old=True
)
```

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
docs[0].page_content
```

```python
vector_db = BaiduVectorDB(embeddings, conn_params)
vector_db.add_texts(["Ankush went to Princeton"])
query = "Where did Ankush go to college?"
docs = vector_db.max_marginal_relevance_search(query)
docs[0].page_content
```