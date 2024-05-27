# 递归 URL

我们可能希望处理根目录下的所有 URL。

例如，让我们来看一下[Python 3.9 文档](https://docs.python.org/3.9/)。

这里有许多有趣的子页面，我们可能希望批量阅读。

当然，`WebBaseLoader` 可以加载页面列表。

但是，挑战在于遍历子页面的树，并实际组装该列表！

我们使用 `RecursiveUrlLoader` 来实现这一点。

这还使我们能够排除一些子页面，自定义提取器等等。

# 参数

- url: str, 要爬取的目标 url。

- exclude_dirs: Optional[str], 要排除的网页目录。

- use_async: Optional[bool], 是否使用异步请求，使用异步请求通常在大型任务中更快。但是，异步将禁用延迟加载功能（函数仍然有效，但不是延迟加载）。默认情况下，设置为 False。

- extractor: Optional[Callable[[str], str]], 从网页中提取文档文本的函数，默认情况下返回页面本身。建议使用类似 goose3 和 beautifulsoup 的工具来提取文本。默认情况下，它只返回页面本身。

- max_depth: Optional[int] = None, 爬取的最大深度。默认为 2。如果需要爬取整个网站，请将其设置为足够大的数字。

- timeout: Optional[int] = None, 每个请求的超时时间，单位为秒。默认为 10。

- prevent_outside: Optional[bool] = None, 是否阻止爬取根 url 之外的内容。默认为 True。

```python
from langchain_community.document_loaders.recursive_url_loader import RecursiveUrlLoader
```

让我们尝试一个简单的例子。

```python
from bs4 import BeautifulSoup as Soup
url = "https://docs.python.org/3.9/"
loader = RecursiveUrlLoader(
    url=url, max_depth=2, extractor=lambda x: Soup(x, "html.parser").text
)
docs = loader.load()
```

```python
docs[0].page_content[:50]
```

```output
'\n\n\n\n\nPython Frequently Asked Questions — Python 3.'
```

```python
docs[-1].metadata
```

```output
{'source': 'https://docs.python.org/3.9/library/index.html',
 'title': 'The Python Standard Library — Python 3.9.17 documentation',
 'language': None}
```

然而，由于很难进行完美的过滤，你可能仍然会在结果中看到一些不相关的内容。如果需要，您可以自行对返回的文档进行过滤。大多数情况下，返回的结果已经足够好了。

在 LangChain 文档上进行测试。

```python
url = "https://js.langchain.com/docs/modules/memory/integrations/"
loader = RecursiveUrlLoader(url=url)
docs = loader.load()
len(docs)
```

```output
8
```