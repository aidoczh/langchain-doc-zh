# Elasticsearch

使用托管的嵌入模型在Elasticsearch中生成嵌入的详细步骤

实例化`ElasticsearchEmbeddings`类的最简单方法是

- 如果您使用Elastic Cloud，则使用`from_credentials`构造函数

- 或者使用`from_es_connection`构造函数与任何Elasticsearch集群一起使用

```python
!pip -q install langchain-elasticsearch
```

```python
from langchain_elasticsearch import ElasticsearchEmbeddings
```

```python
# 定义模型ID
model_id = "your_model_id"
```

## 使用`from_credentials`进行测试

这需要Elastic Cloud的`cloud_id`

```python
# 使用凭据实例化ElasticsearchEmbeddings
embeddings = ElasticsearchEmbeddings.from_credentials(
    model_id,
    es_cloud_id="your_cloud_id",
    es_user="your_user",
    es_password="your_password",
)
```

```python
# 为多个文档创建嵌入
documents = [
    "This is an example document.",
    "Another example document to generate embeddings for.",
]
document_embeddings = embeddings.embed_documents(documents)
```

```python
# 打印文档嵌入
for i, embedding in enumerate(document_embeddings):
    print(f"文档{i+1}的嵌入: {embedding}")
```

```python
# 为单个查询创建嵌入
query = "This is a single query."
query_embedding = embeddings.embed_query(query)
```

```python
# 打印查询嵌入
print(f"查询的嵌入: {query_embedding}")
```

## 使用现有的Elasticsearch客户端连接进行测试

这可以与任何Elasticsearch部署一起使用

```python
# 创建Elasticsearch连接
from elasticsearch import Elasticsearch
es_connection = Elasticsearch(
    hosts=["https://es_cluster_url:port"], basic_auth=("user", "password")
)
```

```python
# 使用es_connection实例化ElasticsearchEmbeddings
embeddings = ElasticsearchEmbeddings.from_es_connection(
    model_id,
    es_connection,
)
```

```python
# 为多个文档创建嵌入
documents = [
    "This is an example document.",
    "Another example document to generate embeddings for.",
]
document_embeddings = embeddings.embed_documents(documents)
```

```python
# 打印文档嵌入
for i, embedding in enumerate(document_embeddings):
    print(f"文档{i+1}的嵌入: {embedding}")
```

```python
# 为单个查询创建嵌入
query = "This is a single query."
query_embedding = embeddings.embed_query(query)
```

```python
# 打印查询嵌入
print(f"查询的嵌入: {query_embedding}")
```