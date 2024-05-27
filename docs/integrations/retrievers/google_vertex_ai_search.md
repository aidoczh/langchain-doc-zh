# 谷歌 Vertex AI 搜索

[谷歌 Vertex AI 搜索](https://cloud.google.com/enterprise-search)（以前称为 `Generative AI App Builder` 上的 `企业搜索`）是由 `Google Cloud` 提供的 `Vertex AI` 机器学习平台的一部分。

`Vertex AI 搜索` 让组织能够快速构建基于生成式人工智能的搜索引擎，面向客户和员工。它基于各种 `Google Search` 技术，包括语义搜索，通过使用自然语言处理和机器学习技术来推断内容内部的关系和用户查询意图，从而比传统基于关键字的搜索技术提供更相关的结果。`Vertex AI 搜索` 还受益于谷歌在理解用户搜索行为和考虑内容相关性方面的专业知识。

`Vertex AI 搜索` 可以在 `Google Cloud Console` 和通过企业工作流集成的 API 中使用。

这个笔记本演示了如何配置 `Vertex AI 搜索` 并使用 `Vertex AI 搜索检索器。` `Vertex AI 搜索检索器` 封装了 [Python 客户端库](https://cloud.google.com/generative-ai-app-builder/docs/libraries#client-libraries-install-python)，并使用它来访问 [搜索服务 API](https://cloud.google.com/python/docs/reference/discoveryengine/latest/google.cloud.discoveryengine_v1beta.services.search_service)。

## 安装先决条件

您需要安装 `google-cloud-discoveryengine` 包才能使用 `Vertex AI 搜索检索器`。

```python
%pip install --upgrade --quiet google-cloud-discoveryengine
```

## 配置访问谷歌云和 Vertex AI 搜索

截至 2023 年 8 月，`Vertex AI 搜索` 已经普遍可用，无需白名单。

在您使用检索器之前，您需要完成以下步骤：

### 创建搜索引擎并填充非结构化数据存储

- 按照 [Vertex AI 搜索入门指南](https://cloud.google.com/generative-ai-app-builder/docs/try-enterprise-search) 中的说明设置谷歌云项目和 Vertex AI 搜索。

- [使用谷歌云控制台创建非结构化数据存储](https://cloud.google.com/generative-ai-app-builder/docs/create-engine-es#unstructured-data)

  - 使用来自 `gs://cloud-samples-data/gen-app-builder/search/alphabet-investor-pdfs` 云存储文件夹的示例 PDF 文档进行填充。

  - 确保使用 `Cloud Storage（不带元数据）` 选项。

### 设置凭据以访问 Vertex AI 搜索 API

`Vertex AI 搜索` 检索器使用的 [Vertex AI 搜索客户端库](https://cloud.google.com/generative-ai-app-builder/docs/libraries) 提供了高级语言支持，用于以编程方式对谷歌云进行身份验证。

客户端库支持 [应用程序默认凭据（ADC）](https://cloud.google.com/docs/authentication/application-default-credentials)；这些库会在一组定义的位置查找凭据，并使用这些凭据对 API 的请求进行身份验证。

使用 ADC，您可以在各种环境中（例如本地开发或生产环境）使凭据对您的应用程序可用，而无需修改应用程序代码。

如果在 [Google Colab](https://colab.research.google.com) 中运行，请使用 `google.colab.google.auth` 进行身份验证，否则请按照 [支持的方法](https://cloud.google.com/docs/authentication/application-default-credentials) 确保您的应用程序默认凭据已正确设置。

```python
import sys
if "google.colab" in sys.modules:
    from google.colab import auth as google_auth
    google_auth.authenticate_user()
```

## 配置和使用 Vertex AI 搜索检索器

`Vertex AI 搜索` 检索器实现在 `langchain.retriever.GoogleVertexAISearchRetriever` 类中。`get_relevant_documents` 方法返回一个 `langchain.schema.Document` 文档列表，其中每个文档的 `page_content` 字段填充了文档内容。

根据在 `Vertex AI 搜索` 中使用的数据类型（网站、结构化或非结构化），`page_content` 字段填充如下：

- 具有高级索引的网站：与查询匹配的 `抽取式答案`。`metadata` 字段填充了从中提取段落或答案的文档的元数据（如果有的话）。

- 非结构化数据源：与查询匹配的 `抽取式段落` 或 `抽取式答案`。`metadata` 字段填充了从中提取段落或答案的文档的元数据（如果有的话）。

- 结构化数据源：一个包含从结构化数据源返回的所有字段的字符串 JSON。`metadata` 字段填充了从中提取段落或答案的文档的元数据（如果有的话）。

### 抽取式答案和抽取式段落

提取式答案是从原始文档中直接提取的逐字文本，它会随每个搜索结果一起返回。通常会在网页顶部附近显示，为最终用户提供与其查询相关的简要答案。提取式答案可用于网站和非结构化搜索。

提取式段落是随每个搜索结果一起返回的逐字文本。提取式段落通常比提取式答案更冗长。提取式段落可以显示为对查询的答案，并可用于执行后处理任务以及作为大型语言模型生成答案或新文本的输入。提取式段落可用于非结构化搜索。

有关提取式段落和提取式答案的更多信息，请参阅[产品文档](https://cloud.google.com/generative-ai-app-builder/docs/snippets)。

注意：提取式段落需要启用[企业版](https://cloud.google.com/generative-ai-app-builder/docs/about-advanced-features#enterprise-features)功能。

在创建检索器实例时，可以指定一些参数，用于控制访问哪个数据存储和如何处理自然语言查询，包括提取式答案和段落的配置。

### 必填参数包括：

- `project_id` - 您的 Google 云项目 ID。

- `location_id` - 数据存储的位置。

  - `global`（默认值）

  - `us`

  - `eu`

其中之一：

- `search_engine_id` - 您要使用的搜索应用的 ID。（对混合搜索来说是必需的）

- `data_store_id` - 您要使用的数据存储的 ID。

`project_id`、`search_engine_id` 和 `data_store_id` 参数可以在检索器的构造函数中明确提供，也可以通过环境变量 `PROJECT_ID`、`SEARCH_ENGINE_ID` 和 `DATA_STORE_ID` 提供。

您还可以配置一些可选参数，包括：

- `max_documents` - 用于提供提取式段落或提取式答案的最大文档数。

- `get_extractive_answers` - 默认情况下，检索器配置为返回提取式段落。

  - 将此字段设置为 `True` 以返回提取式答案。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。

- `max_extractive_answer_count` - 每个搜索结果中返回的提取式答案的最大数量。

  - 最多返回 5 个答案。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。

- `max_extractive_segment_count` - 每个搜索结果中返回的提取式段落的最大数量。

  - 目前将返回一个段落。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。

- `filter` - 基于数据存储中文档的元数据进行搜索结果的过滤表达式。

- `query_expansion_condition` - 用于确定何种条件下进行查询扩展的规范。

  - `0` - 未指定的查询扩展条件。在这种情况下，服务器行为默认为禁用。

  - `1` - 禁用查询扩展。仅使用精确搜索查询，即使 SearchResponse.total_size 为零。

  - `2` - 由搜索 API 构建的自动查询扩展。

- `engine_data_type` - 定义 Vertex AI Search 数据类型

  - `0` - 非结构化数据

  - `1` - 结构化数据

  - `2` - 网站数据

  - `3` - [混合搜索](https://cloud.google.com/generative-ai-app-builder/docs/create-data-store-es#multi-data-stores)

### `GoogleCloudEnterpriseSearchRetriever` 的迁移指南

在以前的版本中，此检索器称为 `GoogleCloudEnterpriseSearchRetriever`。

要升级到新的检索器，请进行以下更改：

- 从以下导入更改为：`from langchain.retrievers import GoogleVertexAISearchRetriever` -> `from langchain.retrievers import GoogleVertexAISearchRetriever`。

- 将所有类引用从 `GoogleCloudEnterpriseSearchRetriever` 更改为 `GoogleVertexAISearchRetriever`。

### 配置并使用用于**非结构化**数据的检索器与提取式段落

```python
from langchain_community.retrievers import (
    GoogleVertexAIMultiTurnSearchRetriever,
    GoogleVertexAISearchRetriever,
)
PROJECT_ID = "<YOUR PROJECT ID>"  # 设置为您的项目 ID
LOCATION_ID = "<YOUR LOCATION>"  # 设置为您的数据存储位置
SEARCH_ENGINE_ID = "<YOUR SEARCH APP ID>"  # 设置为您的搜索应用 ID
DATA_STORE_ID = "<YOUR DATA STORE ID>"  # 设置为您的数据存储 ID
```

```python
retriever = GoogleVertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
)
```

```python
query = "What are Alphabet's Other Bets?"
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置并使用用于**非结构化**数据的检索器与提取式答案

```python
retriever = GoogleVertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    max_extractive_answer_count=3,
    get_extractive_answers=True,
)
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器来处理**结构化**数据

```python
retriever = GoogleVertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    engine_data_type=1,
)
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器来处理带有高级网站索引的**网站**数据

```python
retriever = GoogleVertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    max_extractive_answer_count=3,
    get_extractive_answers=True,
    engine_data_type=2,
)
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器来处理**混合**数据

```python
retriever = GoogleVertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    search_engine_id=SEARCH_ENGINE_ID,
    max_documents=3,
    engine_data_type=3,
)
result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器来进行多轮搜索

[带有跟进的搜索](https://cloud.google.com/generative-ai-app-builder/docs/multi-turn-search)基于生成式人工智能模型，与常规的非结构化数据搜索不同。

```python
retriever = GoogleVertexAIMultiTurnSearchRetriever(
    project_id=PROJECT_ID, location_id=LOCATION_ID, data_store_id=DATA_STORE_ID
)
result = retriever.invoke(query)
for doc in result:
    print(doc)
```