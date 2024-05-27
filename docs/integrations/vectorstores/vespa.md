# Vespa

[Vespa](https://vespa.ai/) 是一个功能齐全的搜索引擎和向量数据库。它支持向量搜索（ANN）、词汇搜索和结构化数据搜索，所有这些功能都可以在同一个查询中使用。

本文介绍了如何将 `Vespa.ai` 用作 LangChain 向量存储的示例。

为了创建向量存储，我们使用 [pyvespa](https://pyvespa.readthedocs.io/en/latest/index.html) 来连接到 `Vespa` 服务。

```python
%pip install --upgrade --quiet  pyvespa
```

使用 `pyvespa` 包，您可以连接到 [Vespa Cloud 实例](https://pyvespa.readthedocs.io/en/latest/deploy-vespa-cloud.html) 或本地 [Docker 实例](https://pyvespa.readthedocs.io/en/latest/deploy-docker.html)。

在这里，我们将创建一个新的 Vespa 应用程序，并使用 Docker 部署。

#### 创建 Vespa 应用程序

首先，我们需要创建一个应用程序包。

```python
from vespa.package import ApplicationPackage, Field, RankProfile
app_package = ApplicationPackage(name="testapp")
app_package.schema.add_fields(
    Field(
        name="text", type="string", indexing=["index", "summary"], index="enable-bm25"
    ),
    Field(
        name="embedding",
        type="tensor<float>(x[384])",
        indexing=["attribute", "summary"],
        attribute=["distance-metric: angular"],
    ),
)
app_package.schema.add_rank_profile(
    RankProfile(
        name="default",
        first_phase="closeness(field, embedding)",
        inputs=[("query(query_embedding)", "tensor<float>(x[384])")],
    )
)
```

这样设置了一个 Vespa 应用程序，其中包含一个模式，每个文档都包含两个字段：`text` 用于保存文档文本，`embedding` 用于保存嵌入向量。`text` 字段设置为使用 BM25 索引以实现高效的文本检索，稍后我们将看到如何使用它和混合搜索。

`embedding` 字段设置为一个长度为 384 的向量，用于保存文本的嵌入表示。有关 Vespa 中张量的更多信息，请参阅 [Vespa 的张量指南](https://docs.vespa.ai/en/tensor-user-guide.html)。

最后，我们添加了一个 [排名配置](https://docs.vespa.ai/en/ranking.html)，指示 Vespa 如何排序文档。在这里，我们使用了 [最近邻搜索](https://docs.vespa.ai/en/nearest-neighbor-search.html)。

现在，我们可以在本地部署这个应用程序。

```python
from vespa.deployment import VespaDocker
vespa_docker = VespaDocker()
vespa_app = vespa_docker.deploy(application_package=app_package)
```

这将部署并创建一个到 `Vespa` 服务的连接。如果您已经有一个 Vespa 应用程序在运行，例如在云中，请参考 PyVespa 应用程序以了解如何连接。

#### 创建 Vespa 向量存储

现在，让我们加载一些文档。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
from langchain_community.embeddings.sentence_transformer import (
    SentenceTransformerEmbeddings,
)
embedding_function = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
```

在这里，我们还设置了一个本地句子嵌入器，将文本转换为嵌入向量。也可以使用 OpenAI 的嵌入，但是向量长度需要更新为 `1536`，以反映该嵌入的更大尺寸。

为了将它们提供给 Vespa，我们需要配置向量存储如何映射到 Vespa 应用程序中的字段。然后，我们直接从这组文档创建向量存储。

```python
vespa_config = dict(
    page_content_field="text",
    embedding_field="embedding",
    input_field="query_embedding",
)
from langchain_community.vectorstores import VespaStore
db = VespaStore.from_documents(docs, embedding_function, app=vespa_app, **vespa_config)
```

这将创建一个 Vespa 向量存储，并将这组文档提供给 Vespa。向量存储会为每个文档调用嵌入函数，并将它们插入到数据库中。

现在，我们可以查询向量存储。

```python
query = "What did the president say about Ketanji Brown Jackson"
results = db.similarity_search(query)
print(results[0].page_content)
```

这将使用上面给定的嵌入函数为查询创建一个表示，并使用它来搜索 Vespa。请注意，这将使用我们在应用程序包中设置的 `default` 排名函数。您可以使用 `similarity_search` 的 `ranking` 参数指定要使用的排名函数。

有关更多信息，请参考 [pyvespa 文档](https://pyvespa.readthedocs.io/en/latest/getting-started-pyvespa.html#Query)。

这涵盖了在LangChain中使用Vespa存储的基本用法。现在你可以返回结果并继续在LangChain中使用它们。

#### 更新文档

除了调用`from_documents`之外，你还可以直接创建向量存储并从中调用`add_texts`来更新文档：

```python
query = "总统对Ketanji Brown Jackson说了什么"
results = db.similarity_search(query)
result = results[0]
result.page_content = "更新：" + result.page_content
db.add_texts([result.page_content], [result.metadata], result.metadata["id"])
results = db.similarity_search(query)
print(results[0].page_content)
```

然而，`pyvespa`库包含了直接操作Vespa内容的方法，你可以直接使用它们。

#### 删除文档

你可以使用`delete`函数删除文档：

```python
result = db.similarity_search(query)
db.delete(["32"])
result = db.similarity_search(query)
```

同样，`pyvespa`连接也包含了删除文档的方法。

### 返回得分

`similarity_search`方法只按相关性顺序返回文档。要检索实际得分：

```python
results = db.similarity_search_with_score(query)
result = results[0]
```

这是使用“all-MiniLM-L6-v2”嵌入模型和余弦距离函数得到的结果（由应用函数中的`angular`参数给出）。

不同的嵌入函数需要不同的距离函数，Vespa需要知道在排序文档时使用哪个距离函数。请参考[距离函数的文档](https://docs.vespa.ai/en/reference/schema-reference.html#distance-metric)以获取更多信息。

### 作为检索器

要将此向量存储用作[LangChain检索器](/docs/how_to#retrievers)，只需调用`as_retriever`函数，这是一个标准的向量存储方法：

```python
db = VespaStore.from_documents(docs, embedding_function, app=vespa_app, **vespa_config)
retriever = db.as_retriever()
query = "总统对Ketanji Brown Jackson说了什么"
results = retriever.invoke(query)
```

这允许从向量存储中进行更一般的、非结构化的检索。

### 元数据

到目前为止的示例中，我们只使用了文本和该文本的嵌入。文档通常包含其他信息，在LangChain中称为元数据。

Vespa可以包含许多具有不同类型的字段，只需将它们添加到应用程序包中：

```python
app_package.schema.add_fields(
    # ...
    Field(name="date", type="string", indexing=["attribute", "summary"]),
    Field(name="rating", type="int", indexing=["attribute", "summary"]),
    Field(name="author", type="string", indexing=["attribute", "summary"]),
    # ...
)
vespa_app = vespa_docker.deploy(application_package=app_package)
```

我们可以在文档中添加一些元数据字段：

```python
# 添加元数据
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"2023-{(i % 12)+1}-{(i % 28)+1}"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["Joe Biden", "Unknown"][min(i, 1)]
```

然后让Vespa向量存储知道这些字段：

```python
vespa_config.update(dict(metadata_fields=["date", "rating", "author"]))
```

现在，在搜索这些文档时，这些字段将被返回。此外，这些字段可以用于过滤：

```python
db = VespaStore.from_documents(docs, embedding_function, app=vespa_app, **vespa_config)
query = "总统对Ketanji Brown Jackson说了什么"
results = db.similarity_search(query, filter="rating > 3")
```

### 自定义查询

如果相似性搜索的默认行为不符合你的要求，你可以自己提供查询。因此，你不需要向向量存储提供所有配置，而是自己编写。

首先，让我们向我们的应用程序添加一个BM25排名函数：

```python
from vespa.package import FieldSet
app_package.schema.add_field_set(FieldSet(name="default", fields=["text"]))
app_package.schema.add_rank_profile(RankProfile(name="bm25", first_phase="bm25(text)"))
vespa_app = vespa_docker.deploy(application_package=app_package)
db = VespaStore.from_documents(docs, embedding_function, app=vespa_app, **vespa_config)
```

然后，根据BM25执行常规文本搜索：

```python
query = "总统对Ketanji Brown Jackson说了什么"
custom_query = {
    "yql": "select * from sources * where userQuery()",
    "query": query,
    "type": "weakAnd",
    "ranking": "bm25",
    "hits": 4,
}
results = db.similarity_search_with_score(query, custom_query=custom_query)
```

### 混合搜索

混合搜索是指同时使用传统的基于词项的搜索（如BM25）和向量搜索，并将结果进行组合。我们需要为Vespa创建一个新的混合搜索的排名配置文件：

```python
app_package.schema.add_rank_profile(
    RankProfile(
        name="hybrid",
        first_phase="log(bm25(text)) + 0.5 * closeness(field, embedding)",
        inputs=[("query(query_embedding)", "tensor<float>(x[384])")],
    )
)
```

在这里，我们将每个文档的得分定义为其BM25得分和距离得分的组合。我们可以使用自定义查询进行查询：

```python
query = "What did the president say about Ketanji Brown Jackson"
query_embedding = embedding_function.embed_query(query)
nearest_neighbor_expression = "{targetHits: 4}nearestNeighbor(embedding, query_embedding)"
custom_query = {
    "yql": f"select * from sources * where {nearest_neighbor_expression} and userQuery()",
    "query": query,
    "type": "weakAnd",
    "input.query(query_embedding)": query_embedding,
    "ranking": "hybrid",
    "hits": 4,
}
results = db.similarity_search_with_score(query, custom_query=custom_query)
```

### Vespa中的原生嵌入器

到目前为止，我们使用Python中的嵌入函数为文本提供嵌入。Vespa原生支持嵌入函数，因此您可以将此计算延迟到Vespa中。其中一个好处是，如果您有大量文档集合，可以使用GPU来嵌入文档。

请参考[Vespa嵌入](https://docs.vespa.ai/en/embedding.html)获取更多信息。

首先，我们需要修改应用程序包：

```python
from vespa.package import Component, Parameter
app_package.components = [
    Component(
        id="hf-embedder",
        type="hugging-face-embedder",
        parameters=[
            Parameter("transformer-model", {"path": "..."}),
            Parameter("tokenizer-model", {"url": "..."}),
        ],
    )
]
Field(
    name="hfembedding",
    type="tensor<float>(x[384])",
    is_document_field=False,
    indexing=["input text", "embed hf-embedder", "attribute", "summary"],
    attribute=["distance-metric: angular"],
)
app_package.schema.add_rank_profile(
    RankProfile(
        name="hf_similarity",
        first_phase="closeness(field, hfembedding)",
        inputs=[("query(query_embedding)", "tensor<float>(x[384])")],
    )
)
```

请参考嵌入文档中的嵌入模型和分词器的文档。请注意，`hfembedding`字段包含使用`hf-embedder`进行嵌入的指令。

现在我们可以使用自定义查询进行查询：

```python
query = "What did the president say about Ketanji Brown Jackson"
nearest_neighbor_expression = "{targetHits: 4}nearestNeighbor(internalembedding, query_embedding)"
custom_query = {
    "yql": f"select * from sources * where {nearest_neighbor_expression}",
    "input.query(query_embedding)": f'embed(hf-embedder, "{query}")',
    "ranking": "internal_similarity",
    "hits": 4,
}
results = db.similarity_search_with_score(query, custom_query=custom_query)
```

请注意，此处的查询包含一个`embed`指令，用于使用与文档相同的模型对查询进行嵌入。

### 近似最近邻搜索

在上述所有示例中，我们使用了精确的最近邻搜索来查找结果。然而，对于大量文档集合来说，这是不可行的，因为必须扫描所有文档才能找到最佳匹配项。为了避免这种情况，我们可以使用[近似最近邻搜索](https://docs.vespa.ai/en/approximate-nn-hnsw.html)。

首先，我们可以更改嵌入字段以创建HNSW索引：

```python
from vespa.package import HNSW
app_package.schema.add_fields(
    Field(
        name="embedding",
        type="tensor<float>(x[384])",
        indexing=["attribute", "summary", "index"],
        ann=HNSW(
            distance_metric="angular",
            max_links_per_node=16,
            neighbors_to_explore_at_insert=200,
        ),
    )
)
```

这将在嵌入数据上创建一个HNSW索引，从而实现高效的搜索。设置好后，我们可以通过将`approximate`参数设置为`True`来轻松使用ANN进行搜索：

```python
query = "What did the president say about Ketanji Brown Jackson"
results = db.similarity_search(query, approximate=True)
```

这涵盖了 LangChain 中 Vespa 向量存储的大部分功能。