# 路径

> [Pathway](https://pathway.com/) 是一个开放的数据处理框架。它允许您轻松开发数据转换管道和与实时数据源以及不断变化的数据一起工作的机器学习应用程序。

这篇笔记展示了如何使用实时的 `Pathway` 数据索引管道与 `Langchain` 配合使用。您可以以与查询常规向量存储库相同的方式从您的链中查询此管道的结果。然而，在幕后，Pathway 在每次数据更改时更新索引，为您提供始终最新的答案。

在这篇笔记中，我们将使用一个[公共演示文档处理管道](https://pathway.com/solutions/ai-pipelines#try-it-out)，该管道：

1. 监视几个云数据源的数据更改。

2. 为数据构建一个向量索引。

要拥有您自己的文档处理管道，请查看[托管服务](https://pathway.com/solutions/ai-pipelines)或[构建您自己的](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/)。

我们将使用一个 `VectorStore` 客户端连接到索引，该客户端实现了 `similarity_search` 函数以检索匹配的文档。

本文档中使用的基本管道允许轻松构建存储在云位置的文件的简单向量索引。然而，Pathway 提供了构建实时数据管道和应用程序所需的一切，包括类似 SQL 的操作，如分组约简和不同数据源之间的连接，基于时间的数据分组和数据窗口化，以及各种连接器。

## 查询数据管道

要实例化和配置客户端，您需要提供文档索引管道的 `url` 或 `host` 和 `port`。在下面的代码中，我们使用一个公开可用的[演示管道](https://pathway.com/solutions/ai-pipelines#try-it-out)，其 REST API 可以在 `https://demo-document-indexing.pathway.stream` 上访问。此演示从[Google Drive](https://drive.google.com/drive/u/0/folders/1cULDv2OaViJBmOfG5WB0oWcgayNrGtVs)和[Sharepoint](https://navalgo.sharepoint.com/sites/ConnectorSandbox/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2FConnectorSandbox%2FShared%20Documents%2FIndexerSandbox&p=true&ga=1)摄取文档，并维护一个检索文档的索引。

```python
from langchain_community.vectorstores import PathwayVectorClient
client = PathwayVectorClient(url="https://demo-document-indexing.pathway.stream")
```

 然后我们可以开始提出查询

```python
query = "What is Pathway?"
docs = client.similarity_search(query)
```

```python
print(docs[0].page_content)
```

 **轮到您了！** [获取您的管道](https://pathway.com/solutions/ai-pipelines) 或上传[新文档](https://chat-realtime-sharepoint-gdrive.demo.pathway.com/)到演示管道，并重新尝试查询！

## 基于文件元数据的过滤

我们支持使用 [jmespath](https://jmespath.org/) 表达式进行文档过滤，例如：

```python
# 仅考虑晚于 Unix 时间戳的源
docs = client.similarity_search(query, metadata_filter="modified_at >= `1702672093`")
# 仅考虑所有者为 'james' 的源
docs = client.similarity_search(query, metadata_filter="owner == `james`")
# 仅考虑路径包含 'repo_readme' 的源
docs = client.similarity_search(query, metadata_filter="contains(path, 'repo_readme')")
# 两个条件的与
docs = client.similarity_search(
    query, metadata_filter="owner == `james` && modified_at >= `1702672093`"
)
# 两个条件的或
docs = client.similarity_search(
    query, metadata_filter="owner == `james` || modified_at >= `1702672093`"
)
```

## 获取索引文件的信息

 `PathwayVectorClient.get_vectorstore_statistics()` 提供了关于向量存储状态的基本统计信息，如索引文件数量和最后更新的时间戳。您可以在您的链中使用它告诉用户您的知识库有多新鲜。

```python
client.get_vectorstore_statistics()
```

## 您自己的管道

### 在生产环境中运行

要拥有您自己的 Pathway 数据索引管道，请查看 Pathway 提供的[托管管道](https://pathway.com/solutions/ai-pipelines)。您也可以运行自己的 Pathway 管道 - 有关如何构建管道的信息，请参考[Pathway 指南](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/)。

### 处理文档

向量化处理流程支持可插拔组件，用于解析、拆分和嵌入文档。对于嵌入和拆分，您可以使用[Langchain组件](https://pathway.com/developers/user-guide/llm-xpack/vectorstore_pipeline/#langchain)，或查看Pathway提供的[嵌入器](https://pathway.com/developers/api-docs/pathway-xpacks-llm/embedders)和[拆分器](https://pathway.com/developers/api-docs/pathway-xpacks-llm/splitters)。如果未提供解析器，则默认使用`UTF-8`解析器。您可以在[这里](https://github.com/pathwaycom/pathway/blob/main/python/pathway/xpacks/llm/parser.py)找到可用的解析器。