# Mojeek 搜索

以下笔记将解释如何使用 Mojeek 搜索获取结果。请访问 [Mojeek 网站](https://www.mojeek.com/services/search/web-search-api/) 获取 API 密钥。

```python
from langchain_community.tools import MojeekSearch
```

```python
api_key = "KEY"  # 从 Mojeek 网站获取
```

```python
search = MojeekSearch.config(api_key=api_key, search_kwargs={"t": 10})
```

在 `search_kwargs` 中，您可以添加任何您在 [Mojeek 文档](https://www.mojeek.com/support/api/search/request_parameters.html) 中找到的搜索参数。

```python
search.run("mojeek")
```