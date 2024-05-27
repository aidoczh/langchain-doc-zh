# RST

> [reStructured Text (RST)](https://en.wikipedia.org/wiki/ReStructuredText) 文件是一种用于文本数据的文件格式，主要在 Python 编程语言社区中用于技术文档。

## `UnstructuredRSTLoader`

您可以使用 `UnstructuredRSTLoader` 从 RST 文件中加载数据，具体操作如下。

```python
from langchain_community.document_loaders import UnstructuredRSTLoader
```

```python
loader = UnstructuredRSTLoader(file_path="example_data/README.rst", mode="elements")
docs = loader.load()
```

```python
print(docs[0])
```

```output
page_content='Example Docs' metadata={'source': 'example_data/README.rst', 'filename': 'README.rst', 'file_directory': 'example_data', 'filetype': 'text/x-rst', 'page_number': 1, 'category': 'Title'}
```