# OpenSearch

[OpenSearch](https://opensearch.org/) 是一个可扩展、灵活、可扩展的开源软件套件，用于搜索、分析和可观察性应用，采用 Apache 2.0 许可。`OpenSearch` 是基于 `Apache Lucene` 的分布式搜索和分析引擎。

这篇笔记展示了如何使用与 `OpenSearch` 数据库相关的功能。

要运行，您应该有一个正在运行的 OpenSearch 实例：[在这里查看如何轻松使用 Docker 进行安装](https://hub.docker.com/r/opensearchproject/opensearch)。

`similarity_search` 默认执行近似 k-NN 搜索，使用 lucene、nmslib、faiss 等多种算法之一，推荐用于大型数据集。要执行暴力搜索，我们有其他搜索方法，称为脚本评分和无痛脚本。

查看[这里](https://opensearch.org/docs/latest/search-plugins/knn/index/)了解更多详情。

## 安装

安装 Python 客户端。

```python
%pip install --upgrade --quiet  opensearch-py
```

我们想要使用 OpenAIEmbeddings，因此我们必须获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import OpenSearchVectorSearch
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

## 使用近似 k-NN 进行相似性搜索

使用自定义参数使用 `Approximate k-NN` 搜索的 `similarity_search`

```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200"
)
# 如果使用默认的 Docker 安装，请改用以下实例化：
# docsearch = OpenSearchVectorSearch.from_documents(
#     docs,
#     embeddings,
#     opensearch_url="https://localhost:9200",
#     http_auth=("admin", "admin"),
#     use_ssl = False,
#     verify_certs = False,
#     ssl_assert_hostname = False,
#     ssl_show_warn = False,
# )
```

```python
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.similarity_search(query, k=10)
```

```python
print(docs[0].page_content)
```

```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="http://localhost:9200",
    engine="faiss",
    space_type="innerproduct",
    ef_construction=256,
    m=48,
)
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.similarity_search(query)
```

```python
print(docs[0].page_content)
```

## 使用脚本评分进行相似性搜索

使用自定义参数使用 `Script Scoring` 的 `similarity_search`

```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200", is_appx_search=False
)
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.similarity_search(
    "总统对 Ketanji Brown Jackson 有什么看法",
    k=1,
    search_type="script_scoring",
)
```

```python
print(docs[0].page_content)
```

## 使用无痛脚本进行相似性搜索

使用自定义参数使用 `Painless Scripting` 的 `similarity_search`

```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200", is_appx_search=False
)
filter = {"bool": {"filter": {"term": {"text": "smuggling"}}}}
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.similarity_search(
    "总统对 Ketanji Brown Jackson 有什么看法",
    search_type="painless_scripting",
    space_type="cosineSimilarity",
    pre_filter=filter,
)
```

```python
print(docs[0].page_content)
```

## 最大边际相关性搜索 (MMR)

如果您想查找一些相似的文档，但又希望获得多样化的结果，MMR 是您应该考虑的方法。最大边际相关性优化了与查询的相似性和所选文档之间的多样性。

```python
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10, lambda_param=0.5)
```

## 使用现有的 OpenSearch 实例

也可以使用已经存在具有向量的文档的现有 OpenSearch 实例。

```python
# 这只是一个示例，您需要更改这些值以指向另一个 opensearch 实例
docsearch = OpenSearchVectorSearch(
    index_name="index-*",
    embedding_function=embeddings,
    opensearch_url="http://localhost:9200",
)
# 您可以指定自定义字段名称以匹配您用于存储嵌入、文档文本值和元数据的字段
docs = docsearch.similarity_search(
    "今天谁在问午餐的事？",
    search_type="script_scoring",
    space_type="cosinesimil",
    vector_field="message_embedding",
    text_field="message",
    metadata_field="message_metadata",
)
```

## 使用 AOSS（Amazon OpenSearch Service 无服务器版）

这是一个使用 `AOSS`、`faiss` 引擎和 `efficient_filter` 的示例。

我们需要安装几个 `python` 包。

```python
%pip install --upgrade --quiet  boto3 requests requests-aws4auth
```

```python
import boto3
from opensearchpy import RequestsHttpConnection
from requests_aws4auth import AWS4Auth
service = "aoss"  # 必须将服务设置为 'aoss'
region = "us-east-2"
credentials = boto3.Session(
    aws_access_key_id="xxxxxx", aws_secret_access_key="xxxxx"
).get_credentials()
awsauth = AWS4Auth("xxxxx", "xxxxxx", region, service, session_token=credentials.token)
docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="host url",
    http_auth=awsauth,
    timeout=300,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection,
    index_name="test-index-using-aoss",
    engine="faiss",
)
docs = docsearch.similarity_search(
    "What is feature selection",
    efficient_filter=filter,
    k=200,
)
```

## 使用 AOS（Amazon OpenSearch Service）

```python
%pip install --upgrade --quiet  boto3
```

```python
# 这只是一个示例，展示如何使用 Amazon OpenSearch Service，您需要设置正确的值。
import boto3
from opensearchpy import RequestsHttpConnection
service = "es"  # 必须将服务设置为 'es'
region = "us-east-2"
credentials = boto3.Session(
    aws_access_key_id="xxxxxx", aws_secret_access_key="xxxxx"
).get_credentials()
awsauth = AWS4Auth("xxxxx", "xxxxxx", region, service, session_token=credentials.token)
docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="host url",
    http_auth=awsauth,
    timeout=300,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection,
    index_name="test-index",
)
docs = docsearch.similarity_search(
    "What is feature selection",
    k=200,
)
```