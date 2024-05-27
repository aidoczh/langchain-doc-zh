# 阿里云 OpenSearch

[阿里云 OpenSearch](https://www.alibabacloud.com/product/opensearch) 是一个一站式平台，用于开发智能搜索服务。`OpenSearch` 是基于 `阿里巴巴` 开发的大规模分布式搜索引擎构建的。`OpenSearch` 为阿里巴巴集团的 500 多个业务案例和数千名阿里云客户提供服务。`OpenSearch` 帮助在不同的搜索场景中开发搜索服务，包括电子商务、O2O、多媒体、内容行业、社区和论坛以及企业中的大数据查询。

`OpenSearch` 帮助您开发高质量、免维护和高性能的智能搜索服务，为用户提供高效和准确的搜索体验。

`OpenSearch` 提供了向量搜索功能。在特定场景中，特别是测试题搜索和图像搜索场景中，您可以结合使用向量搜索功能和多模态搜索功能，以提高搜索结果的准确性。

本笔记本展示了如何使用与 `阿里云 OpenSearch 向量搜索版` 相关的功能。

## 设置

### 购买实例并进行配置

从[阿里云](https://opensearch.console.aliyun.com)购买 OpenSearch 向量搜索版，并根据[文档](https://help.aliyun.com/document_detail/463198.html?spm=a2c4g.465092.0.0.2cd15002hdwavO)进行实例配置。

要运行，您应该已经启动了一个[OpenSearch 向量搜索版](https://opensearch.console.aliyun.com)实例。

### 阿里云 OpenSearch 向量存储类

`AlibabaCloudOpenSearch` 类支持以下功能：

- `add_texts`

- `add_documents`

- `from_texts`

- `from_documents`

- `similarity_search`

- `asimilarity_search`

- `similarity_search_by_vector`

- `asimilarity_search_by_vector`

- `similarity_search_with_relevance_scores`

- `delete_doc_by_texts`

阅读[帮助文档](https://www.alibabacloud.com/help/en/opensearch/latest/vector-search)快速熟悉并配置 OpenSearch 向量搜索版实例。

如果在使用过程中遇到任何问题，请随时联系 xingshaomin.xsm@alibaba-inc.com，我们将尽力为您提供帮助和支持。

实例启动后，按照以下步骤拆分文档、获取嵌入、连接到阿里云 OpenSearch 实例、索引文档并执行向量检索。

首先，我们需要安装以下 Python 包。

```python
%pip install --upgrade --quiet  alibabacloud_ha3engine_vector
```

我们希望使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## 示例

```python
from langchain_community.vectorstores import (
    AlibabaCloudOpenSearch,
    AlibabaCloudOpenSearchSettings,
)
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

拆分文档并获取嵌入。

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../../state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

创建 OpenSearch 设置。

```python
settings = AlibabaCloudOpenSearchSettings(
    endpoint="阿里云 OpenSearch 实例的端点，您可以从阿里云 OpenSearch 控制台中找到。",
    instance_id="阿里云 OpenSearch 实例的标识，您可以从阿里云 OpenSearch 控制台中找到。",
    protocol="SDK 与服务器之间的通信协议，默认为 http。",
    username="购买实例时指定的用户名。",
    password="购买实例时指定的密码。",
    namespace="基于命名空间字段对实例数据进行分区。如果启用了命名空间，您需要在初始化时指定命名空间字段名称。否则，查询将无法正确执行。",
    tablename="在实例配置期间指定的表名。",
    embedding_field_separator="用于编写向量字段数据的分隔符，默认为逗号。",
    output_fields="调用 OpenSearch 时返回的字段列表，默认为字段映射字段的值列表。",
    field_name_mapping={
        "id": "id",  # 索引文档的 id 字段名称映射。
        "document": "document",  # 索引文档的文本字段名称映射。
        "embedding": "embedding",  # 索引文档的嵌入字段名称映射。
        "name_of_the_metadata_specified_during_search": "opensearch_metadata_field_name,=",
        # 索引文档的元数据字段名称映射，可以指定多个。值字段包含映射名称和运算符，在执行元数据过滤查询时将使用运算符。
        # 当前支持的逻辑运算符有：>（大于）、<（小于）、=（等于）、<=（小于或等于）、>=（大于或等于）、！=（不等于）。
        # 参考此链接：https://help.aliyun.com/zh/open-search/vector-search-edition/filter-expression
    },
)
```

例如

```python
settings = AlibabaCloudOpenSearchSettings(
    endpoint='ha-cn-5yd3fhdm102.public.ha.aliyuncs.com',
    instance_id='ha-cn-5yd3fhdm102',
    username='实例用户名',
    password='实例密码',
    table_name='test_table',
    field_name_mapping={
        "id": "id",
        "document": "document",
        "embedding": "embedding",
        "string_field": "string_filed,=",
        "int_field": "int_filed,=",
        "float_field": "float_field,=",
        "double_field": "double_field,="
    },
)
```

通过设置创建一个 opensearch 访问实例。

```python
# 创建一个 opensearch 实例并索引文档。
opensearch = AlibabaCloudOpenSearch.from_texts(
    texts=docs, embedding=embeddings, config=settings
)
```

或者

```python
# 创建一个 opensearch 实例。
opensearch = AlibabaCloudOpenSearch(embedding=embeddings, config=settings)
```

添加文本并构建索引。

```python
metadatas = [
    {"string_field": "value1", "int_field": 1, "float_field": 1.0, "double_field": 2.0},
    {"string_field": "value2", "int_field": 2, "float_field": 3.0, "double_field": 4.0},
    {"string_field": "value3", "int_field": 3, "float_field": 5.0, "double_field": 6.0},
]
# metadatas 的键必须与 settings 中的 field_name_mapping 匹配。
opensearch.add_texts(texts=docs, ids=[], metadatas=metadatas)
```

查询和检索数据。

```python
query = "总统对 Ketanji Brown Jackson 有什么评论"
docs = opensearch.similarity_search(query)
print(docs[0].page_content)
```

带有元数据的查询和检索数据。

```python
query = "总统对 Ketanji Brown Jackson 有什么评论"
metadata = {
    "string_field": "value1",
    "int_field": 1,
    "float_field": 1.0,
    "double_field": 2.0,
}
docs = opensearch.similarity_search(query, filter=metadata)
print(docs[0].page_content)
```

如果在使用过程中遇到任何问题，请随时联系 <xingshaomin.xsm@alibaba-inc.com>，我们将尽力为您提供帮助和支持。

```