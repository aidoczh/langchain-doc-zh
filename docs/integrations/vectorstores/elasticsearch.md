

# Elasticsearch

[Elasticsearch](https://www.elastic.co/elasticsearch/) 是一个分布式的、基于 RESTful 的搜索和分析引擎，能够执行向量搜索和词法搜索。它是构建在 Apache Lucene 库之上。

本文档展示了如何使用与 `Elasticsearch` 数据库相关的功能。

```python
%pip install --upgrade --quiet langchain-elasticsearch langchain-openai tiktoken langchain
```

## 运行和连接到 Elasticsearch

有两种主要方式可以设置 Elasticsearch 实例以供使用：

1. Elastic Cloud：Elastic Cloud 是一个托管的 Elasticsearch 服务。注册免费试用版 [free trial](https://cloud.elastic.co/registration?utm_source=langchain&utm_content=documentation)。

要连接到一个不需要登录凭据的 Elasticsearch 实例（启动具有安全性启用的 Docker 实例），请将 Elasticsearch URL 和索引名称以及嵌入对象传递给构造函数。

2. 本地安装 Elasticsearch：通过在本地运行 Elasticsearch 来开始使用。最简单的方法是使用官方的 Elasticsearch Docker 镜像。有关更多信息，请参阅 [Elasticsearch Docker 文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)。

### 通过 Docker 运行 Elasticsearch

示例：运行一个禁用安全性的单节点 Elasticsearch 实例。这不推荐用于生产环境。

```bash
docker run -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.http.ssl.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.12.1
```

一旦 Elasticsearch 实例运行起来，您可以使用 Elasticsearch URL 和索引名称以及嵌入对象连接到它。

示例：

```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding
)
```

### 认证

对于生产环境，我们建议您启用安全性。要使用登录凭据连接，您可以使用参数 `es_api_key` 或 `es_user` 和 `es_password`。

示例：

```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

您还可以使用一个提供更多灵活性的 `Elasticsearch` 客户端对象，例如配置最大重试次数。

示例：

```python
import elasticsearch
from langchain_elasticsearch import ElasticsearchStore
es_client= elasticsearch.Elasticsearch(
    hosts=["http://localhost:9200"],
    es_user="elastic",
    es_password="changeme",
    max_retries=10,
)
embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    index_name="test_index",
    es_connection=es_client,
    embedding=embedding,
)
```

#### 如何获取默认 "elastic" 用户的密码？

要获取默认 "elastic" 用户的 Elastic Cloud 密码：

1. 登录到 Elastic Cloud 控制台 https://cloud.elastic.co

2. 转到 "Security" > "Users"

3. 找到 "elastic" 用户并点击 "Edit"

4. 点击 "Reset password"

5. 按照提示重置密码

#### 如何获取 API 密钥？

要获取 API 密钥：

1. 登录到 Elastic Cloud 控制台 https://cloud.elastic.co

2. 打开 Kibana 并转到 Stack Management > API Keys

3. 点击 "Create API key"

4. 输入 API 密钥的名称并点击 "Create"

5. 复制 API 密钥并粘贴到 `api_key` 参数中

### Elastic Cloud

要连接到 Elastic Cloud 上的 Elasticsearch 实例，您可以使用 `es_cloud_id` 参数或 `es_url`。

示例：

```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_cloud_id="<cloud_id>",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

要使用 `OpenAIEmbeddings`，我们必须在环境中配置 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## 基本示例

在这个示例中，我们将通过 TextLoader 加载 "state_of_the_union.txt"，将文本分块为 500 个单词的块，然后将每个块索引到 Elasticsearch。

一旦数据被索引，我们执行一个简单的查询，找到与查询 "What did the president say about Ketanji Brown Jackson" 相似的前 4 个块。

Elasticsearch 在本地的 localhost:9200 上运行，使用 [docker](#running-elasticsearch-via-docker)。有关如何从 Elastic Cloud 连接到 Elasticsearch 的更多详细信息，请参见上面的 [connecting with authentication](#authentication)。

```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test-basic",
)
db.client.indices.refresh(index="test-basic")
query = "What did the president say about Ketanji Brown Jackson"
results = db.similarity_search(query)
print(results)
```

```output
[Document(page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='As I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. \n\nWhile it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='This is personal to me and Jill, to Kamala, and to so many of you. \n\nCancer is the #2 cause of death in America–second only to heart disease. \n\nLast month, I announced our plan to supercharge  \nthe Cancer Moonshot that President Obama asked me to lead six years ago. \n\nOur goal is to cut the cancer death rate by at least 50% over the next 25 years, turn more cancers from death sentences into treatable diseases.  \n\nMore support for patients and families.', metadata={'source': '../../how_to/state_of_the_union.txt'})]
```

# 元数据

`ElasticsearchStore` 支持存储与文档一起的元数据。这个元数据字典对象存储在 Elasticsearch 文档中的元数据对象字段中。根据元数据值，Elasticsearch 将自动设置映射，推断元数据值的数据类型。例如，如果元数据值是字符串，Elasticsearch 将设置元数据对象字段的映射为字符串类型。

```python
# 添加元数据到文档
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]
db = ElasticsearchStore.from_documents(
    docs, embeddings, es_url="http://localhost:9200", index_name="test-metadata"
)
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].metadata)
```

```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

## 过滤元数据

在文档中添加了元数据后，您可以在查询时添加元数据过滤。

### 示例：按精确关键词过滤

注意：我们使用了未经分析的关键字子字段

```python
docs = db.similarity_search(
    query, filter=[{"term": {"metadata.author.keyword": "John Doe"}}]
)
print(docs[0].metadata)
```

```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

### 示例：按部分匹配过滤

这个例子展示了如何通过部分匹配进行过滤。当你不知道元数据字段的确切值时，这非常有用。例如，如果你想通过元数据字段`author`进行过滤，但你不知道作者的确切值，你可以使用部分匹配来通过作者的姓氏进行过滤。模糊匹配也是支持的。

"Jon"与"John Doe"匹配，因为"Jon"与"John"是相似的标记。

```python
docs = db.similarity_search(
    query,
    filter=[{"match": {"metadata.author": {"query": "Jon", "fuzziness": "AUTO"}}}],
)
print(docs[0].metadata)
```

```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

### 示例：按日期范围过滤

```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[{"range": {"metadata.date": {"gte": "2010-01-01"}}}],
)
print(docs[0].metadata)
```

```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```

### 示例：按数值范围过滤

```python
docs = db.similarity_search(
    "Any mention about Fred?", filter=[{"range": {"metadata.rating": {"gte": 2}}}]
)
print(docs[0].metadata)
```

```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```

### 示例：按地理距离过滤

需要声明一个映射到`metadata.geo_location`的geo_point索引。

```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[
        {
            "geo_distance": {
                "distance": "200km",
                "metadata.geo_location": {"lat": 40, "lon": -70},
            }
        }
    ],
)
print(docs[0].metadata)
```

过滤器支持比上述更多类型的查询。

在[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)中了解更多信息。

# 距离相似度算法

Elasticsearch支持以下向量距离相似度算法：

- 余弦相似度（cosine）

- 欧几里得距离（euclidean）

- 点积（dot_product）

余弦相似度算法是默认的。

你可以通过相似度参数指定所需的相似度算法。

**注意**

根据检索策略的不同，相似度算法在查询时无法更改。它需要在创建字段的索引映射时设置。如果需要更改相似度算法，需要删除索引并使用正确的distance_strategy重新创建。

```python
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test",
    distance_strategy="COSINE"
    # distance_strategy="EUCLIDEAN_DISTANCE"
    # distance_strategy="DOT_PRODUCT"
)
```

# 检索策略

Elasticsearch相对于其他仅支持向量的数据库具有很大的优势，因为它能够支持各种检索策略。在本文档中，我们将配置`ElasticsearchStore`以支持一些常见的检索策略。

默认情况下，`ElasticsearchStore`使用`DenseVectorStrategy`（在版本0.2.0之前称为`ApproxRetrievalStrategy`）。

## DenseVectorStrategy

这将返回与查询向量最相似的前`k`个向量。`k`参数在初始化`ElasticsearchStore`时设置。默认值为`10`。

```python
from langchain_elasticsearch import DenseVectorStrategy
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test",
    strategy=DenseVectorStrategy(),
)
docs = db.similarity_search(
    query="What did the president say about Ketanji Brown Jackson?", k=10
)
```

### 示例：使用密集向量和关键词搜索进行混合检索

此示例将展示如何配置`ElasticsearchStore`以执行混合检索，使用近似语义搜索和基于关键词的搜索的组合。

我们使用RRF来平衡来自不同检索方法的两个分数。

要启用混合检索，需要在`DenseVectorStrategy`构造函数中设置`hybrid=True`。

```python
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test",
    strategy=DenseVectorStrategy(hybrid=True)
)
```

当启用混合检索时，执行的查询将是近似语义搜索和基于关键词的搜索的组合。

它将使用`rrf`（Reciprocal Rank Fusion）来平衡来自不同检索方法的两个分数。

**注意** RRF需要Elasticsearch 8.9.0或更高版本。

```json
{
    "knn": {
        "field": "vector",
        "filter": [],
        "k": 1,
        "num_candidates": 50,
        "query_vector": [1.0, ..., 0.0],
    },
    "query": {
        "bool": {
            "filter": [],
            "must": [{"match": {"text": {"query": "foo"}}}],
        }
    },
    "rank": {"rrf": {}},
}
```

### 示例：在 Elasticsearch 中使用嵌入模型进行密集向量搜索

本示例将展示如何配置 `ElasticsearchStore` 来使用在 Elasticsearch 中部署的嵌入模型进行密集向量检索。

要使用此功能，请通过 `DenseVectorStrategy` 构造函数的 `query_model_id` 参数指定模型 ID。

**注意**：这需要在 Elasticsearch 的 ml 节点上部署和运行模型。请参阅[笔记本示例](https://github.com/elastic/elasticsearch-labs/blob/main/notebooks/integrations/hugging-face/loading-model-from-hugging-face.ipynb)了解如何使用 eland 部署模型。

```python
DENSE_SELF_DEPLOYED_INDEX_NAME = "test-dense-self-deployed"
# 注意：这里没有指定嵌入函数
# 相反，我们将使用在 Elasticsearch 中部署的嵌入模型
db = ElasticsearchStore(
    es_cloud_id="<你的云 ID>",
    es_user="elastic",
    es_password="<你的密码>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)
# 设置一个 Ingest Pipeline 来对文本字段进行嵌入
db.client.ingest.put_pipeline(
    id="test_pipeline",
    processors=[
        {
            "inference": {
                "model_id": "sentence-transformers__all-minilm-l6-v2",
                "field_map": {"query_field": "text_field"},
                "target_field": "vector_query_field",
            }
        }
    ],
)
# 创建一个新的索引，并使用该 Pipeline 进行处理
# 不依赖于 langchain 来创建索引
db.client.indices.create(
    index=DENSE_SELF_DEPLOYED_INDEX_NAME,
    mappings={
        "properties": {
            "text_field": {"type": "text"},
            "vector_query_field": {
                "properties": {
                    "predicted_value": {
                        "type": "dense_vector",
                        "dims": 384,
                        "index": True,
                        "similarity": "l2_norm",
                    }
                }
            },
        }
    },
    settings={"index": {"default_pipeline": "test_pipeline"}},
)
db.from_texts(
    ["hello world"],
    es_cloud_id="<云 ID>",
    es_user="elastic",
    es_password="<云密码>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)
# 执行搜索
db.similarity_search("hello world", k=10)
```

## SparseVectorStrategy (ELSER)

该策略使用 Elasticsearch 的稀疏向量检索来检索前 k 个结果。目前我们只支持我们自己的 "ELSER" 嵌入模型。

**注意**：这需要在 Elasticsearch 的 ml 节点上部署和运行 ELSER 模型。

要使用此功能，请在 `ElasticsearchStore` 构造函数中指定 `SparseVectorStrategy`（在版本 0.2.0 之前称为 `SparseVectorRetrievalStrategy`）。您需要提供一个模型 ID。

```python
from langchain_elasticsearch import SparseVectorStrategy
# 注意，此示例没有指定嵌入函数。这是因为我们在 Elasticsearch 中的索引时间和查询时间推断出了标记。
# 这需要加载并在 Elasticsearch 中运行 ELSER 模型。
db = ElasticsearchStore.from_documents(
    docs,
    es_cloud_id="<云 ID>",
    es_user="elastic",
    es_password="<云密码>",
    index_name="test-elser",
    strategy=SparseVectorStrategy(model_id=".elser_model_2"),
)
db.client.indices.refresh(index="test-elser")
results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson", k=4
)
print(results[0])
```

```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

## DenseVectorScriptScoreStrategy

该策略使用 Elasticsearch 的脚本评分查询来执行精确向量检索（也称为暴力检索）以检索前 k 个结果。（在版本 0.2.0 之前，该策略称为 `ExactRetrievalStrategy`。）

要使用此功能，请在 `ElasticsearchStore` 构造函数中指定 `DenseVectorScriptScoreStrategy`。

```python
from langchain_elasticsearch import SparseVectorStrategy
db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=DenseVectorScriptScoreStrategy(),
)
```

## BM25Strategy

最后，您可以使用全文关键字搜索。

要使用此功能，请在 `ElasticsearchStore` 构造函数中指定 `BM25Strategy`。

```python
from langchain_elasticsearch import BM25Strategy
db = ElasticsearchStore.from_documents(
    docs, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=BM25Strategy(),
)
```

## BM25RetrievalStrategy

这种策略允许用户使用纯BM25进行搜索，而不使用向量搜索。

要使用这种策略，在`ElasticsearchStore`构造函数中指定`BM25RetrievalStrategy`。

请注意，在下面的示例中，未指定嵌入选项，这表示搜索是在不使用嵌入的情况下进行的。

```python
from langchain_elasticsearch import ElasticsearchStore
db = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    strategy=ElasticsearchStore.BM25RetrievalStrategy(),
)
db.add_texts(
    ["foo", "foo bar", "foo bar baz", "bar", "bar baz", "baz"],
)
results = db.similarity_search(query="foo", k=10)
print(results)
```

```output
[Document(page_content='foo'), Document(page_content='foo bar'), Document(page_content='foo bar baz')]
```

## 自定义查询

通过在搜索中使用`custom_query`参数，您可以调整用于从Elasticsearch检索文档的查询。如果要使用更复杂的查询来支持字段的线性增强，这将非常有用。

```python
# 一个只在文本字段上进行BM25搜索的自定义查询示例。
def custom_query(query_body: dict, query: str):
    """用于在Elasticsearch中使用的自定义查询。
    Args:
        query_body (dict): Elasticsearch查询体。
        query (str): 查询字符串。
    Returns:
        dict: Elasticsearch查询体。
    """
    print("检索策略创建的查询检索器:")
    print(query_body)
    print()
    new_query_body = {"query": {"match": {"text": query}}}
    print("实际在Elasticsearch中使用的查询:")
    print(new_query_body)
    print()
    return new_query_body
results = db.similarity_search(
    "总统对Ketanji Brown Jackson有何评论",
    k=4,
    custom_query=custom_query,
)
print("结果:")
print(results[0])
```

```output
检索策略创建的查询检索器:
{'query': {'bool': {'must': [{'text_expansion': {'vector.tokens': {'model_id': '.elser_model_1', 'model_text': 'What did the president say about Ketanji Brown Jackson'}}}], 'filter': []}}}
实际在Elasticsearch中使用的查询:
{'query': {'match': {'text': 'What did the president say about Ketanji Brown Jackson'}}}
结果:
page_content='总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。\n\n4天前，我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续布雷尔大法官卓越的传统。' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

# 自定义文档构建器

通过在搜索中使用`doc_builder`参数，您可以调整使用从Elasticsearch检索的数据构建文档的方式。如果您有使用Langchain创建的索引，这将非常有用。

```python
from typing import Dict
from langchain_core.documents import Document
def custom_document_builder(hit: Dict) -> Document:
    src = hit.get("_source", {})
    return Document(
        page_content=src.get("content", "缺少内容!"),
        metadata={
            "page_number": src.get("page_number", -1),
            "original_filename": src.get("original_filename", "缺少文件名!"),
        },
    )
results = db.similarity_search(
    "总统对Ketanji Brown Jackson有何评论",
    k=4,
    doc_builder=custom_document_builder,
)
print("结果:")
print(results[0])
```

# 常见问题解答

## 问题：将文档索引到Elasticsearch时出现超时错误。如何解决？

一个可能的问题是您的文档可能需要更长的时间才能索引到Elasticsearch中。ElasticsearchStore使用Elasticsearch批量API，该API有一些默认值，您可以调整以减少超时错误的几率。

当您使用SparseVectorRetrievalStrategy时，这也是一个好主意。

默认值为：

- `chunk_size`: 500

- `max_chunk_bytes`: 100MB

要调整这些值，您可以将`chunk_size`和`max_chunk_bytes`参数传递给ElasticsearchStore的`add_texts`方法。

```python
    vector_store.add_texts(
        texts,
        bulk_kwargs={
            "chunk_size": 50,
            "max_chunk_bytes": 200000000
        }
    )
```

# 升级到ElasticsearchStore

如果您已经在基于 langchain 的项目中使用 Elasticsearch，您可能正在使用旧的实现：`ElasticVectorSearch` 和 `ElasticKNNSearch`，这两者现在已经被弃用。我们引入了一个名为 `ElasticsearchStore` 的新实现，它更加灵活和易于使用。本笔记将指导您完成升级到新实现的过程。

## 新特性

新的实现现在是一个名为 `ElasticsearchStore` 的类，可以用于近似稠密向量、精确稠密向量、稀疏向量（ELSER）、BM25 检索和混合检索，通过策略。

## 我正在使用 ElasticKNNSearch

旧的实现：

```python
from langchain_community.vectorstores.elastic_vector_search import ElasticKNNSearch
db = ElasticKNNSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)
```

新的实现：

```python
from langchain_elasticsearch import ElasticsearchStore, DenseVectorStrategy
db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  # if you use the model_id
  # strategy=DenseVectorStrategy(model_id="test_model")
  # if you use hybrid search
  # strategy=DenseVectorStrategy(hybrid=True)
)
```

## 我正在使用 ElasticVectorSearch

旧的实现：

```python
from langchain_community.vectorstores.elastic_vector_search import ElasticVectorSearch
db = ElasticVectorSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)
```

新的实现：

```python
from langchain_elasticsearch import ElasticsearchStore, DenseVectorScriptScoreStrategy
db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  strategy=DenseVectorScriptScoreStrategy()
)
```

```python
db.client.indices.delete(
    index="test-metadata, test-elser, test-basic",
    ignore_unavailable=True,
    allow_no_indices=True,
)
```

```output
ObjectApiResponse({'acknowledged': True})
```