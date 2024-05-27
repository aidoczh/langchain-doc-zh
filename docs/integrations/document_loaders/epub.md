# EPub

[EPUB](https://en.wikipedia.org/wiki/EPUB) 是一种使用“.epub”文件扩展名的电子书文件格式。该术语缩写自电子出版物，有时写作 ePub。`EPUB` 受到许多电子阅读器的支持，并且大多数智能手机、平板电脑和计算机都有兼容的软件可用。

这里介绍了如何将 `.epub` 文档加载到我们可以在下游使用的文档格式中。您需要安装 [`pandoc`](https://pandoc.org/installing.html) 软件包才能使此加载程序正常工作。

```python
%pip install --upgrade --quiet  pandoc
```

```python
from langchain_community.document_loaders import UnstructuredEPubLoader
```

```python
loader = UnstructuredEPubLoader("winter-sports.epub")
```

```python
data = loader.load()
```

## 保留元素

在幕后，Unstructured 为不同的文本块创建不同的“元素”。默认情况下，我们将它们合并在一起，但您可以通过指定 `mode="elements"` 轻松保留这种分离。

```python
loader = UnstructuredEPubLoader("winter-sports.epub", mode="elements")
```

```python
data = loader.load()
```

```python
data[0]
```

```output
Document(page_content='The Project Gutenberg eBook of Winter Sports in\nSwitzerland, by E. F. Benson', lookup_str='', metadata={'source': 'winter-sports.epub', 'page_number': 1, 'category': 'Title'}, lookup_index=0)
```