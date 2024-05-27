# 开放文档格式（ODT）

[办公应用开放文档格式（ODF）](https://en.wikipedia.org/wiki/OpenDocument)，也称为 `OpenDocument`，是一种用于文字处理文档、电子表格、演示文稿和图形的开放文件格式，使用 ZIP 压缩的 XML 文件。它的开发目的是为办公应用提供一种开放的基于 XML 的文件格式规范。

这一标准是由结构化信息标准促进组织（`OASIS`）的技术委员会开发和维护的。它基于 Sun Microsystems 为 OpenOffice.org XML 制定的规范，这是 `OpenOffice.org` 和 `LibreOffice` 的默认格式。最初是为 `StarOffice` 开发的，旨在为办公文档提供一个开放的标准。

`UnstructuredODTLoader` 用于加载 `Open Office ODT` 文件。

```python
from langchain_community.document_loaders import UnstructuredODTLoader
```

```python
loader = UnstructuredODTLoader("example_data/fake.odt", mode="elements")
docs = loader.load()
docs[0]
```

```output
Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.odt', 'filename': 'example_data/fake.odt', 'category': 'Title'})
```