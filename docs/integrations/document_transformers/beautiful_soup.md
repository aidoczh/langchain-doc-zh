# Beautiful Soup

[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) 是一个用于解析 HTML 和 XML 文档的 Python 包（包括具有格式不正确的标记的文档，即非闭合标记，因此被命名为标记汤）。它为解析页面创建了一个解析树，可用于从 HTML 中提取数据[3]，这对于网页抓取非常有用。

`Beautiful Soup` 提供了对 HTML 内容的精细控制，可以实现特定标记的提取、删除和内容清理。

它适用于需要提取特定信息并根据需求清理 HTML 内容的情况。

例如，我们可以从 HTML 内容中抓取 `<p>, <li>, <div>, and <a>` 标记内的文本内容：

* `<p>`：段落标记。它定义了 HTML 中的段落，用于将相关的句子和/或短语分组在一起。

* `<li>`：列表项标记。它用于有序（`<ol>`）和无序（`<ul>`）列表中定义列表中的各个项目。

* `<div>`：分区标记。它是一个块级元素，用于将其他内联或块级元素分组在一起。

* `<a>`：锚点标记。它用于定义超链接。

```python
from langchain_community.document_loaders import AsyncChromiumLoader
from langchain_community.document_transformers import BeautifulSoupTransformer
# 加载 HTML
loader = AsyncChromiumLoader(["https://www.wsj.com"])
html = loader.load()
```

```python
# 转换
bs_transformer = BeautifulSoupTransformer()
docs_transformed = bs_transformer.transform_documents(
    html, tags_to_extract=["p", "li", "div", "a"]
)
```

```python
docs_transformed[0].page_content[0:500]
```

```output
'Conservative legal activists are challenging Amazon, Comcast and others using many of the same tools that helped kill affirmative-action programs in colleges.1,2099 min read U.S. stock indexes fell and government-bond prices climbed, after Moody’s lowered credit ratings for 10 smaller U.S. banks and said it was reviewing ratings for six larger ones. The Dow industrials dropped more than 150 points.3 min read Penn Entertainment’s Barstool Sportsbook app will be rebranded as ESPN Bet this fall as '
```