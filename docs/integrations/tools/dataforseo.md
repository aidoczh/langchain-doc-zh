# DataForSEO

[DataForSeo](https://dataforseo.com/) 通过 API 提供全面的 SEO 和数字营销数据解决方案。

`DataForSeo API` 从像 `Google`、`Bing`、`Yahoo` 这样的最流行的搜索引擎中检索 `SERP`。它还允许从不同类型的搜索引擎（如 `Maps`、`News`、`Events` 等）获取 SERP。

本笔记演示了如何使用 [DataForSeo API](https://dataforseo.com/apis) 获取搜索引擎结果。

```python
from langchain_community.utilities.dataforseo_api_search import DataForSeoAPIWrapper
```

## 设置 API 凭据

您可以通过在 `DataForSeo` 网站上注册来获取 API 凭据。

```python
import os
os.environ["DATAFORSEO_LOGIN"] = "your_api_access_username"
os.environ["DATAFORSEO_PASSWORD"] = "your_api_access_password"
wrapper = DataForSeoAPIWrapper()
```

`run` 方法将返回以下元素中的第一个结果片段：answer_box、knowledge_graph、featured_snippet、shopping、organic。

```python
wrapper.run("Weather in Los Angeles")
```

## `run` 和 `results` 的区别

`run` 和 `results` 是 `DataForSeoAPIWrapper` 类提供的两种方法。

`run` 方法执行搜索并返回来自 answer box、knowledge graph、featured snippet、shopping 或 organic 结果的第一个结果片段。这些元素按优先级从最高到最低排序。

`results` 方法返回根据包装器中设置的参数配置的 JSON 响应。这允许更灵活地确定要从 API 返回的数据。

## 获取 JSON 格式的结果

您可以自定义要在 JSON 响应中返回的结果类型和字段。您还可以设置要返回的顶部结果的最大计数。

```python
json_wrapper = DataForSeoAPIWrapper(
    json_result_types=["organic", "knowledge_graph", "answer_box"],
    json_result_fields=["type", "title", "description", "text"],
    top_count=3,
)
```

```python
json_wrapper.results("Bill Gates")
```

## 自定义位置和语言

您可以通过向 API 包装器传递附加参数来指定搜索结果的位置和语言。

```python
customized_wrapper = DataForSeoAPIWrapper(
    top_count=10,
    json_result_types=["organic", "local_pack"],
    json_result_fields=["title", "description", "type"],
    params={"location_name": "Germany", "language_code": "en"},
)
customized_wrapper.results("coffee near me")
```

## 自定义搜索引擎

您还可以指定要使用的搜索引擎。

```python
customized_wrapper = DataForSeoAPIWrapper(
    top_count=10,
    json_result_types=["organic", "local_pack"],
    json_result_fields=["title", "description", "type"],
    params={"location_name": "Germany", "language_code": "en", "se_name": "bing"},
)
customized_wrapper.results("coffee near me")
```

## 自定义搜索类型

API 包装器还允许您指定要执行的搜索类型。例如，您可以执行地图搜索。

```python
maps_search = DataForSeoAPIWrapper(
    top_count=10,
    json_result_fields=["title", "value", "address", "rating", "type"],
    params={
        "location_coordinate": "52.512,13.36,12z",
        "language_code": "en",
        "se_type": "maps",
    },
)
maps_search.results("coffee near me")
```

## 与 Langchain 代理集成

您可以使用 `langchain.agents` 模块中的 `Tool` 类将 `DataForSeoAPIWrapper` 与 langchain 代理集成。`Tool` 类封装了代理可以调用的函数。

```python
from langchain_core.tools import Tool
search = DataForSeoAPIWrapper(
    top_count=3,
    json_result_types=["organic"],
    json_result_fields=["title", "description", "type"],
)
tool = Tool(
    name="google-search-answer",
    description="My new answer tool",
    func=search.run,
)
json_tool = Tool(
    name="google-search-json",
    description="My new json tool",
    func=search.results,
)
```