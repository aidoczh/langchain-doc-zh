# Couchbase 

[Couchbase](http://couchbase.com/) 是一款备受赞誉的分布式 NoSQL 云数据库，为您的云端、移动、人工智能和边缘计算应用提供了无与伦比的多功能性、性能、可扩展性和经济价值。Couchbase 通过为开发人员提供编码辅助和向量搜索来支持人工智能。

向量搜索是 Couchbase 中的 [全文搜索服务](https://docs.couchbase.com/server/current/learn/services-and-indexes/services/search-service.html)（搜索服务）的一部分。

本教程将解释如何在 Couchbase 中使用向量搜索。您可以使用 [Couchbase Capella](https://www.couchbase.com/products/capella/) 和您自己管理的 Couchbase 服务器进行操作。

## 安装

```python
%pip install --upgrade --quiet langchain langchain-openai couchbase
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## 导入向量存储和嵌入

```python
from langchain_community.vectorstores import CouchbaseVectorStore
from langchain_openai import OpenAIEmbeddings
```

## 创建 Couchbase 连接对象

我们首先创建一个到 Couchbase 集群的连接，然后将集群对象传递给向量存储。

在这里，我们使用用户名和密码进行连接。您也可以使用任何其他支持的方式连接到您的集群。

有关连接到 Couchbase 集群的更多信息，请查看 [Python SDK 文档](https://docs.couchbase.com/python-sdk/current/hello-world/start-using-sdk.html#connect)。

```python
COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # 或者如果使用 TLS，则为 "couchbases://localhost"
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"
```

```python
from datetime import timedelta
from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions
auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)
# 等待集群准备就绪。
cluster.wait_until_ready(timedelta(seconds=5))
```

现在，我们将在 Couchbase 集群中设置要用于向量搜索的存储桶、作用域和集合名称。

在本示例中，我们使用默认的作用域和集合。

```python
BUCKET_NAME = "testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "_default"
SEARCH_INDEX_NAME = "vector-index"
```

在本教程中，我们将使用 OpenAI 嵌入。

```python
embeddings = OpenAIEmbeddings()
```

## 创建搜索索引

目前，搜索索引需要从 Couchbase Capella 或服务器 UI 或使用 REST 接口创建。

让我们在 testing 存储桶上定义一个名为 `vector-index` 的搜索索引。

在本示例中，让我们在 UI 上的搜索服务中使用导入索引功能。

我们在 `testing` 存储桶的 `_default` 作用域上的 `_default` 集合上定义了一个索引，其中向量字段设置为 `embedding`，具有 1536 维度，并且文本字段设置为 `text`。我们还对文档中的所有字段进行索引和存储，以动态映射来适应不同的文档结构。相似度度量设置为 `dot_product`。

### 如何将索引导入到全文搜索服务中？

- [Couchbase 服务器](https://docs.couchbase.com/server/current/search/import-search-index.html)

    - 点击搜索 -> 添加索引 -> 导入

    - 在导入屏幕中复制以下索引定义

    - 点击创建索引以创建索引。

- [Couchbase Capella](https://docs.couchbase.com/cloud/search/import-search-index.html)

    - 将索引定义复制到一个名为 `index.json` 的新文件中

    - 使用文档中的说明在 Capella 中导入文件

    - 点击创建索引以创建索引。

### 索引定义

```
{
 "name": "vector-index",
 "type": "fulltext-index",
 "params": {
  "doc_config": {
   "docid_prefix_delim": "",
   "docid_regexp": "",
   "mode": "type_field",
   "type_field": "type"
  },
  "mapping": {
   "default_analyzer": "standard",
   "default_datetime_parser": "dateTimeOptional",
   "default_field": "_all",
   "default_mapping": {
    "dynamic": true,
    "enabled": true,
    "properties": {
     "metadata": {
      "dynamic": true,
      "enabled": true
     },
     "embedding": {
      "enabled": true,
      "dynamic": false,
      "fields": [
       {
        "dims": 1536,
        "index": true,
        "name": "embedding",
        "similarity": "dot_product",
        "type": "vector",
        "vector_index_optimized_for": "recall"
       }
      ]
     },
     "text": {
      "enabled": true,
      "dynamic": false,
      "fields": [
       {
        "index": true,
        "name": "text",
        "store": true,
        "type": "text"
       }
      ]
     }
    }
   },
   "default_type": "_default",
   "docvalues_dynamic": false,
   "index_dynamic": true,
   "store_dynamic": true,
   "type_field": "_type"
  },
  "store": {
   "indexType": "scorch",
   "segmentVersion": 16
  }
 },
 "sourceType": "gocbcore",
 "sourceName": "testing",
 "sourceParams": {},
 "planParams": {
  "maxPartitionsPerPIndex": 103,
  "indexPartitions": 10,
  "numReplicas": 0
 }
}
```

要了解如何创建支持向量字段的搜索索引的更多详细信息，请参考文档。

- [Couchbase Capella](https://docs.couchbase.com/cloud/vector-search/create-vector-search-index-ui.html)

- [Couchbase Server](https://docs.couchbase.com/server/current/vector-search/create-vector-search-index-ui.html)

## 创建向量存储

我们使用集群信息和搜索索引名称创建向量存储对象。

```python
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
)
```

### 指定文本和嵌入字段

您可以使用 `text_key` 和 `embedding_key` 字段可选地指定文档的文本和嵌入字段。

```python
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
    text_key="text",
    embedding_key="embedding",
)
```

## 基本向量搜索示例

在此示例中，我们将通过 TextLoader 加载 "state_of_the_union.txt" 文件，使用 CharacterTextSplitter 将文本分块为 500 个字符的块，没有重叠，并将所有这些块索引到 Couchbase。

数据索引完成后，我们执行一个简单的查询，以找到与查询 "What did president say about Ketanji Brown Jackson" 相似的前 4 个块。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
vector_store = CouchbaseVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    index_name=SEARCH_INDEX_NAME,
)
```

```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query)
print(results[0])
```

```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

## 带分数的相似性搜索

您可以通过调用 `similarity_search_with_score` 方法获取结果的分数。

```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search_with_score(query)
document, score = results[0]
print(document)
print(f"Score: {score}")
```

```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
Score: 0.8211871385574341
```

## 指定要返回的字段

您可以使用搜索中的 `fields` 参数指定要从文档返回的字段。这些字段作为返回的文档的 `metadata` 对象的一部分返回。您可以获取存储在搜索索引中的任何字段。文档的 `text_key` 作为文档的 `page_content` 的一部分返回。

如果您不指定要获取的任何字段，则将返回索引中存储的所有字段。

如果要获取元数据中的某个字段，需要使用 `.` 指定它。

例如，要获取元数据中的 `source` 字段，需要指定 `metadata.source`。

```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query, fields=["metadata.source"])
print(results[0])
```

## 混合搜索

Couchbase 允许您通过将向量搜索结果与文档的非向量字段（如 `metadata` 对象）的搜索结合起来进行混合搜索。

结果将基于向量搜索和搜索服务支持的搜索结果的组合。每个组件搜索的分数相加以获得结果的总分。

要执行混合搜索，可以将可选参数 `search_options` 传递给所有相似性搜索。

`search_options` 的不同搜索/查询可能性可以在[此处](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object)找到。

### 为混合搜索创建多样化的元数据

为了模拟混合搜索，让我们从现有文档中创建一些随机元数据。

我们均匀地向元数据添加三个字段，`date` 在 2010 年至 2020 年之间，`rating` 在 1 到 5 之间，`author` 设置为 John Doe 或 Jane Doe。

```python
# 为文档添加元数据
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]
vector_store.add_documents(docs)
query = "关于Ketanji Brown Jackson总统说了什么"
results = vector_store.similarity_search(query)
print(results[0].metadata)
```

```output
{'author': 'John Doe', 'date': '2016-01-01', 'rating': 2, 'source': '../../how_to/state_of_the_union.txt'}
```

### 示例：按精确值搜索

我们可以在`metadata`对象中的作者字段上进行精确匹配搜索。

```python
query = "关于Ketanji Brown Jackson总统说了什么"
results = vector_store.similarity_search(
    query,
    search_options={"query": {"field": "metadata.author", "match": "John Doe"}},
    fields=["metadata.author"],
)
print(results[0])
```

```output
page_content='这对我、吉尔、卡玛拉以及你们许多人来说都是个人问题。\n\n癌症是美国的第二大死因，仅次于心脏病。\n\n上个月，我宣布了我们的计划，要加速奥巴马总统六年前要求我领导的癌症登月计划。\n\n我们的目标是在未来25年内将癌症死亡率至少降低50%，将更多的癌症从死刑变为可治疗的疾病。\n\n为患者和家庭提供更多支持。' metadata={'author': 'John Doe'}
```

### 示例：按部分匹配搜索

我们可以通过为搜索指定模糊度来进行部分匹配搜索。当您想要搜索搜索查询的轻微变化或拼写错误时，这非常有用。

在这个例子中，"Jae"与"Jane"接近（模糊度为1）。

```python
query = "关于Ketanji Brown Jackson总统说了什么"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {"field": "metadata.author", "match": "Jae", "fuzziness": 1}
    },
    fields=["metadata.author"],
)
print(results[0])
```

```output
page_content='曾在私人执业中担任高级诉讼律师。曾是联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。是一个共识建设者。自从她被提名以来，她得到了广泛的支持，包括来自民主党和共和党任命的前法官和警察协会。\n\n如果我们要推进自由和正义，我们需要保护边境并修复移民制度。' metadata={'author': 'Jane Doe'}
```

### 示例：按日期范围查询搜索

我们可以在日期字段`metadata.date`上进行日期范围查询的文档搜索。

```python
query = "有关独立的任何提及？"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {
            "start": "2016-12-31",
            "end": "2017-01-02",
            "inclusive_start": True,
            "inclusive_end": False,
            "field": "metadata.date",
        }
    },
)
print(results[0])
```

```output
page_content='他永远不会扼杀他们对自由的热爱。他永远不会削弱自由世界的决心。\n\n我们今晚在一个经历了这个国家有史以来最艰难的两年的美国相遇。\n\n这场大流行病是惩罚性的。\n\n很多家庭每月都靠发薪水为生，努力跟上食品、汽油、住房等费用的上涨。\n\n我理解。' metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}
```

### 示例：按数值范围查询搜索

我们可以在数值字段`metadata.rating`上进行范围查询的文档搜索。

```python
query = "有关独立的任何提及？"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "min": 3,
            "max": 5,
            "inclusive_min": True,
            "inclusive_max": True,
            "field": "metadata.rating",
        }
    },
)
print(results[0])
```

```output
(Document(page_content='他永远不会扼杀他们对自由的热爱。他永远不会削弱自由世界的决心。\n\n我们今晚在一个经历了这个国家有史以来最艰难的两年的美国相遇。\n\n这场大流行病是惩罚性的。\n\n很多家庭每月都靠发薪水为生，努力跟上食品、汽油、住房等费用的上涨。\n\n我理解。', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 0.9000703597577832)
```

### 示例：组合多个搜索查询

不同的搜索查询可以使用AND（合取）或OR（析取）运算符进行组合。

在这个例子中，我们正在检查评级在3和4之间，日期在2015年和2018年之间的文档。

```python
query = "有关独立的任何提及？"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "conjuncts": [
                {"min": 3, "max": 4, "inclusive_max": True, "field": "metadata.rating"},
                {"start": "2016-12-31", "end": "2017-01-02", "field": "metadata.date"},
            ]
        }
    },
)
print(results[0])
```output

(Document(page_content='他永远不会扼杀他们对自由的热爱。他永远不会削弱自由世界的决心。\n\n我们今晚聚集在一个经历了这个国家有史以来最艰难的两年的美国。\n\n这场大流行病是惩罚性的。\n\n许多家庭每个月都在挣扎，努力跟上食品、汽油、住房等成本的上涨。\n\n我理解。', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 1.3598770370389914)

```
### 其他查询
同样，您可以在 `search_options` 参数中使用任何支持的查询方法，如地理距离、多边形搜索、通配符、正则表达式等。请参考文档，了解可用的查询方法及其语法的更多细节。
- [Couchbase Capella](https://docs.couchbase.com/cloud/search/search-request-params.html#query-object)
- [Couchbase Server](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object)
# 常见问题
## 问题：在创建 CouchbaseVectorStore 对象之前，我应该先创建搜索索引吗？
是的，目前在创建 `CouchbaseVectoreStore` 对象之前，您需要先创建搜索索引。
## 问题：我在搜索结果中没有看到我指定的所有字段。
在Couchbase中，我们只能返回搜索索引中存储的字段。请确保您要访问的字段是搜索索引的一部分。
处理此问题的一种方法是在索引中动态地索引和存储文档的字段。
- 在 Capella 中，您需要进入“高级模式”，然后在“常规设置”下的 Chevron 中，您可以选中“[X] 存储动态字段”或“[X] 索引动态字段”
- 在 Couchbase Server 中，在索引编辑器（而非快速编辑器）中的 Chevron “高级”下，您可以选中“[X] 存储动态字段”或“[X] 索引动态字段”
请注意，这些选项将增加索引的大小。
有关动态映射的更多详细信息，请参考[文档](https://docs.couchbase.com/cloud/search/customize-index.html)。
## 问题：我无法在搜索结果中看到 metadata 对象。
这很可能是由于 Couchbase 搜索索引未对文档中的 `metadata` 字段进行索引和/或存储造成的。为了在索引中索引 `metadata` 字段，您需要将其添加到索引中作为子映射。
如果选择映射所有映射中的字段，您将能够搜索所有 metadata 字段。或者，为了优化索引，您可以选择索引 metadata 对象内的特定字段。您可以参考[文档](https://docs.couchbase.com/cloud/search/customize-index.html)了解有关索引子映射的更多信息。
创建子映射
* [Couchbase Capella](https://docs.couchbase.com/cloud/search/create-child-mapping.html)
* [Couchbase Server](https://docs.couchbase.com/server/current/search/create-child-mapping.html)
```