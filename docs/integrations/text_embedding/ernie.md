# ERNIE

[ERNIE Embedding-V1](https://cloud.baidu.com/doc/WENXINWORKSHOP/s/alj562vvu) 是基于百度文心大规模模型技术的文本表示模型，将文本转换为由数值表示的向量形式，可用于文本检索、信息推荐、知识挖掘等场景。

**已弃用警告**

我们建议使用 `langchain_community.embeddings.QianfanEmbeddingsEndpoint` 替代 `langchain_community.embeddings.ErnieEmbeddings`。

`QianfanEmbeddingsEndpoint` 的文档在[这里](/docs/integrations/text_embedding/baidu_qianfan_endpoint/)。

我们推荐用户使用 `QianfanEmbeddingsEndpoint` 的原因有两个：

1. `QianfanEmbeddingsEndpoint` 在千帆平台上支持更多的嵌入模型。

2. `ErnieEmbeddings` 缺乏维护且已弃用。

迁移的一些建议：

```python
from langchain_community.embeddings import QianfanEmbeddingsEndpoint
embeddings = QianfanEmbeddingsEndpoint(
    qianfan_ak="your qianfan ak",
    qianfan_sk="your qianfan sk",
)
```

## 使用方法

```python
from langchain_community.embeddings import ErnieEmbeddings
```

```python
embeddings = ErnieEmbeddings()
```

```python
query_result = embeddings.embed_query("foo")
```

```python
doc_results = embeddings.embed_documents(["foo"])
```