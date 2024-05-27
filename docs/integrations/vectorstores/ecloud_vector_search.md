# 中国移动ECloud ElasticSearch VectorSearch

[中国移动ECloud VectorSearch](https://ecloud.10086.cn/portal/product/elasticsearch) 是一项完全托管的企业级分布式搜索和分析服务。中国移动ECloud VectorSearch为结构化/非结构化数据提供低成本、高性能和可靠的检索和分析平台级产品服务。作为矢量数据库，它支持多种索引类型和相似度距离方法。

本笔记展示了如何使用与`ECloud ElasticSearch VectorStore`相关的功能。

要运行，您应该已经启动了[中国移动ECloud VectorSearch](https://ecloud.10086.cn/portal/product/elasticsearch)实例：

阅读[帮助文档](https://ecloud.10086.cn/op-help-center/doc/category/1094) 快速熟悉并配置中国移动ECloud ElasticSearch实例。

实例启动后，请按照以下步骤拆分文档、获取嵌入、连接到百度云Elasticsearch实例、索引文档并执行矢量检索。

```python
#!pip install elasticsearch == 7.10.1
```

首先，我们想要使用`OpenAIEmbeddings`，因此我们需要获取OpenAI API密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

其次，拆分文档并获取嵌入。

```python
from langchain.vectorstores import EcloudESVectorStore
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../../state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
ES_URL = "http://localhost:9200"
USER = "your user name"
PASSWORD = "your password"
indexname = "your index name"
```

然后，索引文档。

```python
docsearch = EcloudESVectorStore.from_documents(
    docs,
    embeddings,
    es_url=ES_URL,
    user=USER,
    password=PASSWORD,
    index_name=indexname,
    refresh_indices=True,
)
```

最后，查询和检索数据。

```python
query = "总统对Ketanji Brown Jackson有何看法"
docs = docsearch.similarity_search(query, k=10)
print(docs[0].page_content)
```

常用案例

```python
def test_dense_float_vectore_lsh_cosine() -> None:
    """
    Test indexing with vectore type knn_dense_float_vector and  model-similarity of lsh-cosine
    this mapping is compatible with model of exact and similarity of l2/cosine
    this mapping is compatible with model of lsh and similarity of cosine
    """
    docsearch = EcloudESVectorStore.from_documents(
        docs,
        embeddings,
        es_url=ES_URL,
        user=USER,
        password=PASSWORD,
        index_name=indexname,
        refresh_indices=True,
        text_field="my_text",
        vector_field="my_vec",
        vector_type="knn_dense_float_vector",
        vector_params={"model": "lsh", "similarity": "cosine", "L": 99, "k": 1},
    )
    docs = docsearch.similarity_search(
        query,
        k=10,
        search_params={
            "model": "exact",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
    docs = docsearch.similarity_search(
        query,
        k=10,
        search_params={
            "model": "exact",
            "similarity": "l2",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
    docs = docsearch.similarity_search(
        query,
        k=10,
        search_params={
            "model": "exact",
            "similarity": "cosine",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
    docs = docsearch.similarity_search(
        query,
        k=10,
        search_params={
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 10,
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
```

带过滤器的案例

```python
def test_dense_float_vectore_exact_with_filter() -> None:
    """
    Test indexing with vectore type knn_dense_float_vector and default model/similarity
    this mapping is compatible with model of exact and similarity of l2/cosine
    """
    docsearch = EcloudESVectorStore.from_documents(
        docs,
        embeddings,
        es_url=ES_URL,
        user=USER,
        password=PASSWORD,
        index_name=indexname,
        refresh_indices=True,
        text_field="my_text",
        vector_field="my_vec",
        vector_type="knn_dense_float_vector",
    )
    # filter={"match_all": {}} ,default
    docs = docsearch.similarity_search(
        query,
        k=10,
        filter={"match_all": {}},
        search_params={
            "model": "exact",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
    # filter={"term": {"my_text": "Jackson"}}
    docs = docsearch.similarity_search(
        query,
        k=10,
        filter={"term": {"my_text": "Jackson"}},
        search_params={
            "model": "exact",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
    # filter={"term": {"my_text": "president"}}
    docs = docsearch.similarity_search(
        query,
        k=10,
        filter={"term": {"my_text": "president"}},
        search_params={
            "model": "exact",
            "similarity": "l2",
            "vector_field": "my_vec",
            "text_field": "my_text",
        },
    )
    print(docs[0].page_content)
```

抱歉，你似乎忘记输入要翻译的英文段落了。请将需要翻译的英文内容发送给我，我会尽快帮你翻译成中文科普风格的文章。