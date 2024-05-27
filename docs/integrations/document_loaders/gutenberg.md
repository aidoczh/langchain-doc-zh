# 古腾堡计划

[古腾堡计划](https://www.gutenberg.org/about/)是一个提供免费电子书的在线图书馆。

这篇笔记介绍了如何将`古腾堡计划`的电子书链接加载到我们可以在下游使用的文档格式中。

```python
from langchain_community.document_loaders import GutenbergLoader
```

```python
loader = GutenbergLoader("https://www.gutenberg.org/cache/epub/69972/pg69972.txt")
```

```python
data = loader.load()
```

```python
data[0].page_content[:300]
```

```output
'《改变的新娘》，作者 Emma Dorothy\r\n\n\nEliza Nevitte Southworth 的古腾堡计划电子书\r\n\n\n\r\n\n\n本电子书可供美国境内和\r\n\n\n世界大部分其他地区的任何人免费使用，几乎没有任何限制\r\n\n\n。您可以复制、赠送或重新使用'
```

```python
data[0].metadata
```

```output
{'source': 'https://www.gutenberg.org/cache/epub/69972/pg69972.txt'}
```