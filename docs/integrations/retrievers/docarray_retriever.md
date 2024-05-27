# DocArray

[DocArray](https://github.com/docarray/docarray) 是一个多模态数据管理的多功能开源工具。它允许您根据需要塑造数据，并提供了使用各种文档索引后端存储和搜索数据的灵活性。此外，它变得更加强大 - 您可以利用 `DocArray` 文档索引来创建 `DocArrayRetriever`，并构建出色的 Langchain 应用程序！

这篇笔记分为两个部分。[第一部分](#document-index-backends)介绍了所有五种支持的文档索引后端。它提供了设置和索引每个后端的指导，并指导您如何构建 `DocArrayRetriever` 来查找相关文档。

在[第二部分](#movie-retrieval-using-hnswdocumentindex)，我们将选择其中一种后端，并演示如何通过一个基本示例来使用它。

## 文档索引后端

```python
import random
from docarray import BaseDoc
from docarray.typing import NdArray
from langchain_community.embeddings import FakeEmbeddings
from langchain_community.retrievers import DocArrayRetriever
embeddings = FakeEmbeddings(size=32)
```

在开始构建索引之前，定义文档模式非常重要。这决定了您的文档将具有哪些字段，以及每个字段将保存什么类型的数据。

在这个演示中，我们将创建一个包含 'title' (str), 'title_embedding' (numpy array), 'year' (int), 和 'color' (str) 的相对随机的模式。

```python
class MyDoc(BaseDoc):
    title: str
    title_embedding: NdArray[32]
    year: int
    color: str
```

### InMemoryExactNNIndex

`InMemoryExactNNIndex` 将所有文档存储在内存中。对于小数据集来说，这是一个很好的起点，您可能不想启动数据库服务器。

了解更多信息：https://docs.docarray.org/user_guide/storing/index_in_memory/

```python
from docarray.index import InMemoryExactNNIndex
# 初始化索引
db = InMemoryExactNNIndex[MyDoc]()
# 索引数据
db.index(
    [
        MyDoc(
            title=f"My document {i}",
            title_embedding=embeddings.embed_query(f"query {i}"),
            year=i,
            color=random.choice(["red", "green", "blue"]),
        )
        for i in range(100)
    ]
)
# 可选地，您可以创建一个过滤查询
filter_query = {"year": {"$lte": 90}}
```
```python
# 创建一个检索器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="title_embedding",
    content_field="title",
    filters=filter_query,
)
# 查找相关文档
doc = retriever.invoke("some query")
print(doc)
```
```output
[Document(page_content='My document 56', metadata={'id': '1f33e58b6468ab722f3786b96b20afe6', 'year': 56, 'color': 'red'})]
```

### HnswDocumentIndex

`HnswDocumentIndex` 是一个轻量级的文档索引实现，完全在本地运行，最适合中小型数据集。它在 [hnswlib](https://github.com/nmslib/hnswlib) 中存储向量，并在 [SQLite](https://www.sqlite.org/index.html) 中存储所有其他数据。

了解更多信息：https://docs.docarray.org/user_guide/storing/index_hnswlib/

```python
from docarray.index import HnswDocumentIndex
# 初始化索引
db = HnswDocumentIndex[MyDoc](work_dir="hnsw_index")
# 索引数据
db.index(
    [
        MyDoc(
            title=f"My document {i}",
            title_embedding=embeddings.embed_query(f"query {i}"),
            year=i,
            color=random.choice(["red", "green", "blue"]),
        )
        for i in range(100)
    ]
)
# 可选地，您可以创建一个过滤查询
filter_query = {"year": {"$lte": 90}}
```
```python
# 创建一个检索器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="title_embedding",
    content_field="title",
    filters=filter_query,
)
# 查找相关文档
doc = retriever.invoke("some query")
print(doc)
```
```output
[Document(page_content='My document 28', metadata={'id': 'ca9f3f4268eec7c97a7d6e77f541cb82', 'year': 28, 'color': 'red'})]
```

### WeaviateDocumentIndex

`WeaviateDocumentIndex` 是建立在 [Weaviate](https://weaviate.io/) 向量数据库之上的文档索引。

了解更多信息：https://docs.docarray.org/user_guide/storing/index_weaviate/

```python
# 与其他后端相比，Weaviate 后端有一个小差异。
# 在这里，您需要使用 'is_embedding=True' 标记用于向量搜索的字段。
# 因此，让我们为 Weaviate 创建一个新的模式，以满足这个要求。
from pydantic import Field
class WeaviateDoc(BaseDoc):
    title: str
    title_embedding: NdArray[32] = Field(is_embedding=True)
    year: int
    color: str
```
```python
from docarray.index import WeaviateDocumentIndex
# 初始化索引
dbconfig = WeaviateDocumentIndex.DBConfig(host="http://localhost:8080")
db = WeaviateDocumentIndex[WeaviateDoc](db_config=dbconfig)
# 索引数据
db.index(
    [
        MyDoc(
            title=f"My document {i}",
            title_embedding=embeddings.embed_query(f"query {i}"),
            year=i,
            color=random.choice(["red", "green", "blue"]),
        )
        for i in range(100)
    ]
)
# 可选地，您可以创建一个过滤查询
filter_query = {"path": ["year"], "operator": "LessThanEqual", "valueInt": "90"}
```
```python
# 创建一个检索器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="title_embedding",
    content_field="title",
    filters=filter_query,
)
# 查找相关文档
doc = retriever.invoke("一些查询")
print(doc)
```
```output
[Document(page_content='我的文档17', metadata={'id': '3a5b76e85f0d0a01785dc8f9d965ce40', 'year': 17, 'color': '红色'})]
```

### ElasticDocIndex

`ElasticDocIndex` 是建立在 [ElasticSearch](https://github.com/elastic/elasticsearch) 基础上的文档索引

了解更多，请点击[这里](https://docs.docarray.org/user_guide/storing/index_elastic/)

```python
from docarray.index import ElasticDocIndex
# 初始化索引
db = ElasticDocIndex[MyDoc](
    hosts="http://localhost:9200", index_name="docarray_retriever"
)
# 索引数据
db.index(
    [
        MyDoc(
            title=f"我的文档 {i}",
            title_embedding=embeddings.embed_query(f"查询 {i}"),
            year=i,
            color=random.choice(["红色", "绿色", "蓝色"]),
        )
        for i in range(100)
    ]
)
# 可选地，您可以创建一个过滤查询
filter_query = {"range": {"year": {"lte": 90}}}
```
```python
# 创建一个检索器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="title_embedding",
    content_field="title",
    filters=filter_query,
)
# 查找相关文档
doc = retriever.invoke("一些查询")
print(doc)
```
```output
[Document(page_content='我的文档46', metadata={'id': 'edbc721bac1c2ad323414ad1301528a4', 'year': 46, 'color': '绿色'})]
```

### QdrantDocumentIndex

`QdrantDocumentIndex` 是建立在 [Qdrant](https://qdrant.tech/) 向量数据库上的文档索引

了解更多，请点击[这里](https://docs.docarray.org/user_guide/storing/index_qdrant/)

```python
from docarray.index import QdrantDocumentIndex
from qdrant_client.http import models as rest
# 初始化索引
qdrant_config = QdrantDocumentIndex.DBConfig(path=":memory:")
db = QdrantDocumentIndex[MyDoc](qdrant_config)
# 索引数据
db.index(
    [
        MyDoc(
            title=f"我的文档 {i}",
            title_embedding=embeddings.embed_query(f"查询 {i}"),
            year=i,
            color=random.choice(["红色", "绿色", "蓝色"]),
        )
        for i in range(100)
    ]
)
# 可选地，您可以创建一个过滤查询
filter_query = rest.Filter(
    must=[
        rest.FieldCondition(
            key="year",
            range=rest.Range(
                gte=10,
                lt=90,
            ),
        )
    ]
)
```
```output
警告:根据本地 Qdrant 的情况，有效载荷索引没有效果。如果您需要有效载荷索引，请使用服务器 Qdrant。
```
```python
# 创建一个检索器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="title_embedding",
    content_field="title",
    filters=filter_query,
)
# 查找相关文档
doc = retriever.invoke("一些查询")
print(doc)
```
```output
[Document(page_content='我的文档80', metadata={'id': '97465f98d0810f1f330e4ecc29b13d20', 'year': 80, 'color': '蓝色'})]
```

## 使用 HnswDocumentIndex 进行电影检索

```python
movies = [
    {
        "title": "盗梦空间",
        "description": "一名小偷通过梦境共享技术窃取公司机密，被赋予了在CEO的脑海中植入想法的任务。",
        "director": "克里斯托弗·诺兰",
        "评分": 8.8,
    },
    {
        "title": "黑暗骑士",
        "description": "当被称为小丑的威胁对哥谭市的人民造成混乱和混乱时，蝙蝠侠必须接受他对打击不义的能力进行的最大的心理和身体测试之一。",
        "director": "克里斯托弗·诺兰",
        "评分": 9.0,
    },
    {
        "title": "星际穿越",
        "description": "《星际穿越》探索了人类探索的边界，一群宇航员通过太空中的虫洞冒险。在他们的追求中，他们面对着时空的广阔和爱与牺牲的挑战。",
        "director": "克里斯托弗·诺兰",
        "评分": 8.6,
    },
    {
        "title": "低俗小说",
        "description": "两名黑帮杀手、一名拳击手、一名黑帮妻子和一对餐馆抢劫犯的生活在暴力和救赎的四个故事中交织在一起。",
        "director": "昆汀·塔伦蒂诺",
        "评分": 8.9,
    },
    {
        "title": "无间道",
        "description": "当一次简单的珠宝抢劫出了大问题时，幸存的罪犯开始怀疑其中有人是警方线人。",
        "director": "昆汀·塔伦蒂诺",
        "评分": 8.3,
    },
    {
        "title": "教父",
```python

movies = [

    {

        "description": "一个有组织犯罪王朝的老大将帝国的控制权转让给他不情愿的儿子。",

        "director": "弗朗西斯·福特·科波拉",

        "rating": 9.2,

    },

]

```
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output

OpenAI API Key: ········

```
```python
from docarray import BaseDoc, DocList
from docarray.typing import NdArray
from langchain_openai import OpenAIEmbeddings
# 为电影文档定义模式
class MyDoc(BaseDoc):
    title: str
    description: str
    description_embedding: NdArray[1536]
    rating: float
    director: str
embeddings = OpenAIEmbeddings()
# 获取“description”嵌入，并创建文档
docs = DocList[MyDoc](
    [
        MyDoc(
            description_embedding=embeddings.embed_query(movie["description"]), **movie
        )
        for movie in movies
    ]
)
```
```python
from docarray.index import HnswDocumentIndex
# 初始化索引
db = HnswDocumentIndex[MyDoc](work_dir="movie_search")
# 添加数据
db.index(docs)
```
### 普通召回器
```python
from langchain_community.retrievers import DocArrayRetriever
# 创建一个召回器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="description_embedding",
    content_field="description",
)
# 查找相关文档
doc = retriever.invoke("关于梦想的电影")
print(doc)
```
```output

[Document(page_content='一个小偷通过梦境共享技术窃取公司机密，被委托在CEO的脑海中植入一个想法。', metadata={'id': 'f1649d5b6776db04fec9a116bbb6bbe5', 'title': '盗梦空间', 'rating': 8.8, 'director': '克里斯托弗·诺兰'})]

```
### 带过滤器的召回器
```python
from langchain_community.retrievers import DocArrayRetriever
# 创建一个召回器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="description_embedding",
    content_field="description",
    filters={"director": {"$eq": "克里斯托弗·诺兰"}},
    top_k=2,
)
# 查找相关文档
docs = retriever.invoke("太空旅行")
print(docs)
```
```output

[Document(page_content='《星际穿越》探索人类探索的边界，一群宇航员穿越太空中的虫洞。在确保人类生存的过程中，他们面对着时空的广袤，并且探讨了爱与牺牲。', metadata={'id': 'ab704cc7ae8573dc617f9a5e25df022a', 'title': '星际穿越', 'rating': 8.6, 'director': '克里斯托弗·诺兰'}), Document(page_content='一个小偷通过梦境共享技术窃取公司机密，被委托在CEO的脑海中植入一个想法。', metadata={'id': 'f1649d5b6776db04fec9a116bbb6bbe5', 'title': '盗梦空间', 'rating': 8.8, 'director': '克里斯托弗·诺兰'})]

```
### 带 MMR 搜索的召回器
```python
from langchain_community.retrievers import DocArrayRetriever
# 创建一个召回器
retriever = DocArrayRetriever(
    index=db,
    embeddings=embeddings,
    search_field="description_embedding",
    content_field="description",
    filters={"rating": {"$gte": 8.7}},
    search_type="mmr",
    top_k=3,
)
# 查找相关文档
docs = retriever.invoke("动作电影")
print(docs)
```
```output

[Document(page_content='两名黑帮杀手、一名拳击手、一名黑帮头目的妻子和一对餐馆抢劫犯的生活在四个暴力与救赎的故事中交织在一起。', metadata={'id': 'e6aa313bbde514e23fbc80ab34511afd', 'title': '低俗小说', 'rating': 8.9, 'director': '昆汀·塔伦蒂诺'}), Document(page_content='一个小偷通过梦境共享技术窃取公司机密，被委托在CEO的脑海中植入一个想法。', metadata={'id': 'f1649d5b6776db04fec9a116bbb6bbe5', 'title': '盗梦空间', 'rating': 8.8, 'director': '克里斯托弗·诺兰'}), Document(page_content='当被称为小丑的威胁对哥谭市的人民造成混乱和破坏时，蝙蝠侠必须接受他打击不义的心理和身体能力的最大考验之一。', metadata={'id': '91dec17d4272041b669fd113333a65f7', 'title': '黑暗骑士', 'rating': 9.0, 'director': '克里斯托弗·诺兰'})]

```