# Azure AI 搜索

[Azure AI 搜索](https://learn.microsoft.com/azure/search/search-what-is-azure-search)（以前称为 `Azure Cognitive Search`）是微软的云搜索服务，为开发人员提供基础架构、API 和工具，用于大规模检索向量、关键字和混合查询的信息。

`AzureAISearchRetriever` 是一个集成模块，用于从非结构化查询中返回文档。它基于 BaseRetriever 类，并针对 Azure AI 搜索的 2023-11-01 稳定 REST API 版本，这意味着它支持向量索引和查询。

要使用此模块，您需要：

+ 一个 Azure AI 搜索服务。您可以免费[创建一个](https://learn.microsoft.com/azure/search/search-create-service-portal)，如果您注册 Azure 试用账户。免费服务有较低的配额，但足以运行本笔记本中的代码。

+ 具有向量字段的现有索引。有几种创建索引的方法，包括使用[向量存储模块](../vectorstores/azuresearch.md)。或者，[尝试 Azure AI 搜索 REST API](https://learn.microsoft.com/azure/search/search-get-started-vector)。

+ 一个 API 密钥。在创建搜索服务时生成 API 密钥。如果只是查询索引，可以使用查询 API 密钥，否则使用管理员 API 密钥。有关详细信息，请参阅[查找您的 API 密钥](https://learn.microsoft.com/azure/search/search-security-api-keys?tabs=rest-use%2Cportal-find%2Cportal-query#find-existing-keys)。

`AzureAISearchRetriever` 取代了 `AzureCognitiveSearchRetriever`，后者将很快被弃用。我们建议切换到基于最新稳定版本搜索 API 的新版本。

## 安装软件包

使用 azure-documents-search 软件包 11.4 或更高版本。

```python
%pip install --upgrade --quiet langchain
%pip install --upgrade --quiet langchain-openai
%pip install --upgrade --quiet azure-search-documents
%pip install --upgrade --quiet azure-identity
```

## 导入所需库

```python
import os
from langchain_community.retrievers import (
    AzureAISearchRetriever,
)
```

## 配置搜索设置

将搜索服务名称、索引名称和 API 密钥设置为环境变量（或者，您可以将它们作为参数传递给 `AzureAISearchRetriever`）。搜索索引提供可搜索的内容。

```python
os.environ["AZURE_AI_SEARCH_SERVICE_NAME"] = "<YOUR_SEARCH_SERVICE_NAME>"
os.environ["AZURE_AI_SEARCH_INDEX_NAME"] = "<YOUR_SEARCH_INDEX_NAME>"
os.environ["AZURE_AI_SEARCH_API_KEY"] = "<YOUR_API_KEY>"
```

## 创建检索器

对于 `AzureAISearchRetriever`，提供一个 `index_name`、`content_key` 和 `top_k`，设置为您想要检索的结果数量。将 `top_k` 设置为零（默认值）将返回所有结果。

```python
retriever = AzureAISearchRetriever(
    content_key="content", top_k=1, index_name="langchain-vector-demo"
)
```

现在您可以使用它从 Azure AI 搜索中检索文档。

这是您调用的方法。它将返回与查询相关的所有文档。

```python
retriever.invoke("here is my unstructured query string")
```

## 示例

本节演示在内置示例数据上使用检索器。如果您的搜索服务已经具有向量索引，则可以跳过此步骤。

首先提供端点和密钥。由于我们在此步骤中创建一个向量索引，指定一个文本嵌入模型以获得文本的向量表示。此示例假定使用 Azure OpenAI，并部署了 text-embedding-ada-002。因为此步骤创建索引，请确保为您的搜索服务使用管理员 API 密钥。

```python
import os
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain_community.retrievers import AzureAISearchRetriever
from langchain_community.vectorstores import AzureSearch
from langchain_openai import AzureOpenAIEmbeddings, OpenAIEmbeddings
from langchain_text_splitters import TokenTextSplitter
os.environ["AZURE_AI_SEARCH_SERVICE_NAME"] = "<YOUR_SEARCH_SERVICE_NAME>"
os.environ["AZURE_AI_SEARCH_INDEX_NAME"] = "langchain-vector-demo"
os.environ["AZURE_AI_SEARCH_API_KEY"] = "<YOUR_SEARCH_SERVICE_ADMIN_API_KEY>"
azure_endpoint: str = "<YOUR_AZURE_OPENAI_ENDPOINT>"
azure_openai_api_key: str = "<YOUR_AZURE_OPENAI_API_KEY>"
azure_openai_api_version: str = "2023-05-15"
azure_deployment: str = "text-embedding-ada-002"
```

我们将使用 Azure OpenAI 的嵌入模型将我们的文档转换为存储在 Azure AI 搜索向量存储中的嵌入。我们还将将索引名称设置为 `langchain-vector-demo`。这将创建一个与该索引名称关联的新向量存储。

```python
embeddings = AzureOpenAIEmbeddings(
    model=azure_deployment,
    azure_endpoint=azure_endpoint,
    openai_api_key=azure_openai_api_key,
)
vector_store: AzureSearch = AzureSearch(
    embedding_function=embeddings.embed_query,
    azure_search_endpoint=os.getenv("AZURE_AI_SEARCH_SERVICE_NAME"),
    azure_search_key=os.getenv("AZURE_AI_SEARCH_API_KEY"),
    index_name="langchain-vector-demo",
)
```

接下来，我们将把数据加载到我们新创建的向量存储中。在这个示例中，我们加载 `state_of_the_union.txt` 文件。我们将把文本分割成400个标记的块，没有重叠。最后，将这些文档作为嵌入添加到我们的向量存储中。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt", encoding="utf-8")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=400, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
vector_store.add_documents(documents=docs)
```

接下来，我们将创建一个检索器。当前的 `index_name` 变量是上一步中的 `langchain-vector-demo`。如果你跳过了向量存储的创建，可以在参数中提供你的索引名称。在这个查询中，返回顶部结果。

```python
retriever = AzureAISearchRetriever(
    content_key="content", top_k=1, index_name="langchain-vector-demo"
)
```

现在我们可以从我们上传的文档中检索与我们查询相关的数据。

```python
retriever.invoke("does the president have a plan for covid-19?")
```