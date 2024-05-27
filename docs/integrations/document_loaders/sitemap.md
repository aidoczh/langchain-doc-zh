# 网站地图

`SitemapLoader` 是基于 `WebBaseLoader` 的扩展，用于从给定的 URL 加载网站地图，然后并发地进行页面抓取和加载，将每个页面作为一个文档返回。

抓取过程是并发进行的。对于并发请求有合理的限制，默认为每秒 2 次。如果你不关心成为一个良好的网络公民，或者你控制着被抓取的服务器，或者不在意负载。请注意，虽然这样可以加快抓取过程，但可能会导致服务器屏蔽你。请小心操作！

```python
%pip install --upgrade --quiet  nest_asyncio
```

```python
# 修复 asyncio 和 jupyter 的 bug
import nest_asyncio
nest_asyncio.apply()
```

```python
from langchain_community.document_loaders.sitemap import SitemapLoader
```

```python
sitemap_loader = SitemapLoader(web_path="https://api.python.langchain.com/sitemap.xml")
docs = sitemap_loader.load()
```

你可以修改 `requests_per_second` 参数来增加最大并发请求数，并使用 `requests_kwargs` 来传递请求时的关键字参数。

```python
sitemap_loader.requests_per_second = 2
# 可选：避免 `[SSL: CERTIFICATE_VERIFY_FAILED]` 问题
sitemap_loader.requests_kwargs = {"verify": False}
```

```python
docs[0]
```

```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API 参考文档。\n\n\n您将被自动重定向到此页面的新位置。\n\n', metadata={'source': 'https://api.python.langchain.com/en/stable/', 'loc': 'https://api.python.langchain.com/en/stable/', 'lastmod': '2024-02-09T01:10:49.422114+00:00', 'changefreq': 'weekly', 'priority': '1'})
```

## 过滤网站地图 URL

网站地图可能是庞大的文件，包含数千个 URL。通常你并不需要每一个。你可以通过将字符串列表或正则表达式模式传递给 `filter_urls` 参数来过滤 URL。只有匹配其中一个模式的 URL 才会被加载。

```python
loader = SitemapLoader(
    web_path="https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest"],
)
documents = loader.load()
```

```python
documents[0]
```

```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API 参考文档。\n\n\n您将被自动重定向到此页面的新位置。\n\n', metadata={'source': 'https://api.python.langchain.com/en/latest/', 'loc': 'https://api.python.langchain.com/en/latest/', 'lastmod': '2024-02-12T05:26:10.971077+00:00', 'changefreq': 'daily', 'priority': '0.9'})
```

## 添加自定义抓取规则

`SitemapLoader` 使用 `beautifulsoup4` 进行抓取， 默认情况下会抓取页面上的每个元素。`SitemapLoader` 构造函数接受一个自定义抓取函数。这个功能可以帮助你根据特定需求定制抓取过程；例如，你可能想避免抓取标题或导航元素。

以下示例展示了如何开发和使用一个自定义函数来避免抓取导航和标题元素。

导入 `beautifulsoup4` 库并定义自定义函数。

```python
pip install beautifulsoup4
```

```python
from bs4 import BeautifulSoup
def remove_nav_and_header_elements(content: BeautifulSoup) -> str:
    # 在 BeautifulSoup 对象中查找所有 'nav' 和 'header' 元素
    nav_elements = content.find_all("nav")
    header_elements = content.find_all("header")
    # 从 BeautifulSoup 对象中移除每个 'nav' 和 'header' 元素
    for element in nav_elements + header_elements:
        element.decompose()
    return str(content.get_text())
```

将自定义函数添加到 `SitemapLoader` 对象中。

```python
loader = SitemapLoader(
    "https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest/"],
    parsing_function=remove_nav_and_header_elements,
)
```

## 本地网站地图

网站地图加载器也可以用于加载本地文件。

```python
sitemap_loader = SitemapLoader(web_path="example_data/sitemap.xml", is_local=True)
docs = sitemap_loader.load()
```