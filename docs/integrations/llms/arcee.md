# 阿尔茜（Arcee）

本笔记本演示了如何使用 `Arcee` 类来生成文本，使用阿尔茜的领域自适应语言模型（DALMs）。

### 设置

在使用阿尔茜之前，请确保将阿尔茜 API 密钥设置为 `ARCEE_API_KEY` 环境变量。您也可以将 API 密钥作为命名参数传递。

```python
from langchain_community.llms import Arcee
# 创建 Arcee 类的实例
arcee = Arcee(
    model="DALM-PubMed",
    # arcee_api_key="ARCEE-API-KEY" # 如果在环境中尚未设置
)
```

### 额外配置

您还可以根据需要配置阿尔茜的参数，如 `arcee_api_url`、`arcee_app_url` 和 `model_kwargs`。在对象初始化时设置 `model_kwargs` 将使用这些参数作为所有后续调用生成响应的默认值。

```python
arcee = Arcee(
    model="DALM-Patent",
    # arcee_api_key="ARCEE-API-KEY", # 如果在环境中尚未设置
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

### 生成文本

您可以通过提供提示从阿尔茜生成文本。以下是一个示例：

```python
# 生成文本
prompt = "Can AI-driven music therapy contribute to the rehabilitation of patients with disorders of consciousness?"
response = arcee(prompt)
```

### 额外参数

阿尔茜允许您应用 `filters` 并设置检索到的文档的 `size`（按数量计算）以帮助生成文本。过滤器有助于缩小结果范围。以下是如何使用这些参数的示例：

```python
# 定义过滤器
filters = [
    {"field_name": "document", "filter_type": "fuzzy_search", "value": "Einstein"},
    {"field_name": "year", "filter_type": "strict_search", "value": "1905"},
]
# 使用过滤器和大小参数生成文本
response = arcee(prompt, size=5, filters=filters)
```