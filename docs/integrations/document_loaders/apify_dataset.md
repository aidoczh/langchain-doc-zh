# Apify 数据集

[Apify 数据集](https://docs.apify.com/platform/storage/dataset) 是一个可扩展的追加存储系统，支持顺序访问，专为存储结构化的网络爬虫结果而构建，例如产品列表或 Google SERP，然后可以将其导出为 JSON、CSV 或 Excel 等各种格式。数据集主要用于保存 [Apify Actors](https://apify.com/store) 的结果，这些是用于各种网络爬虫、爬取和数据提取用例的无服务器云程序。

这篇笔记展示了如何将 Apify 数据集加载到 LangChain 中。

## 先决条件

您需要在 Apify 平台上拥有一个现有数据集。如果没有，请首先查看[此笔记本](/docs/integrations/tools/apify)，了解如何使用 Apify 从文档、知识库、帮助中心或博客中提取内容。

```python
%pip install --upgrade --quiet  apify-client
```

首先，在您的源代码中导入 `ApifyDatasetLoader`：

```python
from langchain_community.document_loaders import ApifyDatasetLoader
from langchain_core.documents import Document
```

然后提供一个函数，将 Apify 数据集记录字段映射到 LangChain `Document` 格式。

例如，如果您的数据集项结构如下：

```json
{
    "url": "https://apify.com",
    "text": "Apify is the best web scraping and automation platform."
}
```

下面代码中的映射函数将把它们转换为 LangChain `Document` 格式，这样您就可以进一步与任何 LLM 模型（例如用于问答的模型）一起使用。

```python
loader = ApifyDatasetLoader(
    dataset_id="your-dataset-id",
    dataset_mapping_function=lambda dataset_item: Document(
        page_content=dataset_item["text"], metadata={"source": dataset_item["url"]}
    ),
)
```

```python
data = loader.load()
```

## 问答示例

在这个示例中，我们使用数据集中的数据来回答一个问题。

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.document_loaders import ApifyDatasetLoader
from langchain_core.documents import Document
```

```python
loader = ApifyDatasetLoader(
    dataset_id="your-dataset-id",
    dataset_mapping_function=lambda item: Document(
        page_content=item["text"] or "", metadata={"source": item["url"]}
    ),
)
```

```python
index = VectorstoreIndexCreator().from_loaders([loader])
```

```python
query = "What is Apify?"
result = index.query_with_sources(query)
```

```python
print(result["answer"])
print(result["sources"])
```

```output
 Apify is a platform for developing, running, and sharing serverless cloud programs. It enables users to create web scraping and automation tools and publish them on the Apify platform.
https://docs.apify.com/platform/actors, https://docs.apify.com/platform/actors/running/actors-in-store, https://docs.apify.com/platform/security, https://docs.apify.com/platform/actors/examples
```