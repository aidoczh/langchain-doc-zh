# Arcee

[Arcee](https://www.arcee.ai/about/about-us) 有助于开发 SLMs——小型、专业化、安全且可扩展的语言模型。

这篇笔记演示了如何使用 `ArceeRetriever` 类来检索与 Arcee 的 `Domain Adapted Language Models` (`DALMs`) 相关的文档。

### 设置

在使用 `ArceeRetriever` 之前，请确保 Arcee API 密钥已设置为 `ARCEE_API_KEY` 环境变量。您也可以将 API 密钥作为命名参数传递。

```python
from langchain_community.retrievers import ArceeRetriever
retriever = ArceeRetriever(
    model="DALM-PubMed",
    # arcee_api_key="ARCEE-API-KEY" # 如果尚未在环境中设置
)
```

### 附加配置

您还可以根据需要配置 `ArceeRetriever` 的参数，例如 `arcee_api_url`、`arcee_app_url` 和 `model_kwargs`。在对象初始化时设置 `model_kwargs` 将默认使用过滤器和大小进行所有后续检索。

```python
retriever = ArceeRetriever(
    model="DALM-PubMed",
    # arcee_api_key="ARCEE-API-KEY", # 如果尚未在环境中设置
    arcee_api_url="https://custom-api.arcee.ai",  # 默认为 https://api.arcee.ai
    arcee_app_url="https://custom-app.arcee.ai",  # 默认为 https://app.arcee.ai
    model_kwargs={
        "size": 5,
        "filters": [
            {
                "field_name": "document",
                "filter_type": "fuzzy_search",
                "value": "Einstein",
            }
        ],
    },
)
```

### 检索文档

您可以通过提供查询来从上传的上下文中检索相关文档。以下是一个示例：

```python
query = "Can AI-driven music therapy contribute to the rehabilitation of patients with disorders of consciousness?"
documents = retriever.invoke(query)
```

### 附加参数

Arcee 允许您应用 `filters` 并设置检索文档的 `size`（按数量）。过滤器有助于缩小结果范围。以下是如何使用这些参数：

```python
# 定义过滤器
filters = [
    {"field_name": "document", "filter_type": "fuzzy_search", "value": "Music"},
    {"field_name": "year", "filter_type": "strict_search", "value": "1905"},
]
# 使用过滤器和大小参数检索文档
documents = retriever.invoke(query, size=5, filters=filters)
```