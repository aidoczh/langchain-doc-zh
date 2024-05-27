# 腾讯云 VectorDB

[Tencent Cloud VectorDB](https://cloud.tencent.com/document/product/1709) 是一种全托管的、自主开发的企业级分布式数据库服务，旨在存储、检索和分析多维向量数据。该数据库支持多种索引类型和相似度计算方法。单个索引可支持高达10亿的向量规模，并支持数百万的 QPS 和毫秒级的查询延迟。腾讯云 Vector Database 不仅可以为大型模型提供外部知识库，提高大型模型响应的准确性，还可以广泛应用于推荐系统、自然语言处理服务、计算机视觉和智能客服等人工智能领域。

本笔记展示了如何使用与腾讯向量数据库相关的功能。

要运行，请先拥有[数据库实例](https://cloud.tencent.com/document/product/1709/95101)。

## 基本用法

```python
!pip3 install tcvectordb
```
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import TencentVectorDB
from langchain_community.vectorstores.tencentvectordb import ConnectionParams
from langchain_text_splitters import CharacterTextSplitter
```

加载文档，将其分割成块。

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

我们支持两种文档嵌入方式：

- 使用与 Langchain Embeddings 兼容的任何嵌入模型。

- 指定腾讯 VectorStore DB 的嵌入模型名称，可选项包括：

    - `bge-base-zh`，维度：768

    - `m3e-base`，维度：768

    - `text2vec-large-chinese`，维度：1024

    - `e5-large-v2`，维度：1024

    - `multilingual-e5-base`，维度：768

以下代码展示了嵌入文档的两种方式，您可以通过注释另一种方式来选择其中一种：

```python
## 您可以使用 Langchain Embeddings 模型，如 OpenAIEmbeddings：
# from langchain_community.embeddings.openai import OpenAIEmbeddings
# embeddings = OpenAIEmbeddings()
# t_vdb_embedding = None
## 或者您可以使用腾讯嵌入模型，如 `bge-base-zh`：
t_vdb_embedding = "bge-base-zh"  # bge-base-zh 是默认模型
embeddings = None
```

现在我们可以创建一个 TencentVectorDB 实例，您必须至少提供 `embeddings` 或 `t_vdb_embedding` 参数中的一个。如果两者都提供，则将使用 `embeddings` 参数：

```python
conn_params = ConnectionParams(
    url="http://10.0.X.X",
    key="eC4bLRy2va******************************",
    username="root",
    timeout=20,
)
vector_db = TencentVectorDB.from_documents(
    docs, embeddings, connection_params=conn_params, t_vdb_embedding=t_vdb_embedding
)
```
```python
query = "总统关于 Ketanji Brown Jackson 说了什么"
docs = vector_db.similarity_search(query)
docs[0].page_content
```
```output
'今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。而且，顺便说一句，通过《揭示法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。\n\n总统担负的最严肃的宪法责任之一是提名某人担任美国最高法院法官。\n\n而我在4天前就这样做了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。'
```
```python
vector_db = TencentVectorDB(embeddings, conn_params)
vector_db.add_texts(["Ankush 去了普林斯顿大学"])
query = "Ankush 去了哪所大学？"
docs = vector_db.max_marginal_relevance_search(query)
docs[0].page_content
```
```output
'Ankush 去了普林斯顿大学'
```

## 元数据和过滤

腾讯 VectorDB 支持元数据和[过滤](https://cloud.tencent.com/document/product/1709/95099#c6f6d3a3-02c5-4891-b0a1-30fe4daf18d8)。您可以向文档添加元数据，并根据元数据对搜索结果进行过滤。

现在我们将创建一个带有元数据的新 TencentVectorDB 集合，并演示如何根据元数据对搜索结果进行过滤：

```python
from langchain_community.vectorstores.tencentvectordb import (
    META_FIELD_TYPE_STRING,
    META_FIELD_TYPE_UINT64,
    ConnectionParams,
    MetaField,
    TencentVectorDB,
)
from langchain_core.documents import Document
meta_fields = [
    MetaField(name="year", data_type=META_FIELD_TYPE_UINT64, index=True),
    MetaField(name="rating", data_type=META_FIELD_TYPE_STRING, index=False),
    MetaField(name="genre", data_type=META_FIELD_TYPE_STRING, index=True),
    MetaField(name="director", data_type=META_FIELD_TYPE_STRING, index=True),
]
docs = [
    Document(
        page_content="《肖申克的救赎》是一部1994年由弗兰克·德拉邦特编剧并执导的美国戏剧电影。",
        metadata={
            "year": 1994,
            "rating": "9.3",
            "genre": "戏剧",
            "director": "弗兰克·德拉邦特",
        },
    ),
    Document(
        page_content="《教父》是一部1972年由弗朗西斯·福特·科波拉执导的美国犯罪电影。",
        metadata={
            "year": 1972,
            "rating": "9.2",
            "genre": "犯罪",
            "director": "弗朗西斯·福特·科波拉",
        },
    ),
    Document(
        page_content="《黑暗骑士》是一部2008年由克里斯托弗·诺兰执导的超级英雄电影。",
        metadata={
            "year": 2008,
            "rating": "9.0",
            "genre": "超级英雄",
            "director": "克里斯托弗·诺兰",
        },
    ),
    Document(
        page_content="《盗梦空间》是一部2010年由克里斯托弗·诺兰编剧并执导的科幻动作电影。",
        metadata={
            "year": 2010,
            "rating": "8.8",
            "genre": "科幻",
            "director": "克里斯托弗·诺兰",
        },
    ),
]
vector_db = TencentVectorDB.from_documents(
    docs,
    None,
    connection_params=ConnectionParams(
        url="http://10.0.X.X",
        key="eC4bLRy2va******************************",
        username="root",
        timeout=20,
    ),
    collection_name="movies",
    meta_fields=meta_fields,
)
query = "克里斯托弗·诺兰执导的关于梦境的电影"
# 您可以使用 `expr` 参数使用 tencentvectordb 过滤语法：
result = vector_db.similarity_search(query, expr='director="克里斯托弗·诺兰"')
# 您也可以使用 `filter` 参数使用 langchain 过滤语法：
# result = vector_db.similarity_search(query, filter='eq("director", "克里斯托弗·诺兰")')
result
```
```

```

《蝙蝠侠：黑暗骑士》是一部由克里斯托弗·诺兰执导的超级英雄电影，于2008年上映。

《蝙蝠侠：黑暗骑士》是一部由克里斯托弗·诺兰执导的超级英雄电影，于2008年上映。

《蝙蝠侠：黑暗骑士》是一部由克里斯托弗·诺兰执导的超级英雄电影，于2008年上映。

《全面启动》是一部由克里斯托弗·诺兰编剧并执导的科幻动作电影，于2010年上映。