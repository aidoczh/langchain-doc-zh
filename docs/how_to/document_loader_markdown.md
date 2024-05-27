# 如何加载 Markdown

[Markdown](https://en.wikipedia.org/wiki/Markdown) 是一种轻量级标记语言，可用于使用纯文本编辑器创建格式化文本。

在这里，我们将介绍如何将 `Markdown` 文档加载到 LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document) 对象中，以便在下游使用。

我们将介绍：

- 基本用法；

- 将 Markdown 解析为标题、列表项和文本等元素。

LangChain 实现了一个 [UnstructuredMarkdownLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.markdown.UnstructuredMarkdownLoader.html) 对象，它需要使用 [Unstructured](https://unstructured-io.github.io/unstructured/) 包。首先我们需要安装它：

```python
# !pip install "unstructured[md]"
```

基本用法将会将一个 Markdown 文件加载到单个文档中。这里我们演示了在 LangChain 的 readme 上的操作：

```python
from langchain_community.document_loaders import UnstructuredMarkdownLoader
from langchain_core.documents import Document
markdown_path = "../../../../README.md"
loader = UnstructuredMarkdownLoader(markdown_path)
data = loader.load()
assert len(data) == 1
assert isinstance(data[0], Document)
readme_content = data[0].page_content
print(readme_content[:250])
```

```output
🦜️🔗 LangChain
⚡ Build context-aware reasoning applications ⚡
Looking for the JS/TS library? Check out LangChain.js.
To help you ship LangChain apps to production faster, check out LangSmith. 
LangSmith is a unified developer platform for building,
```

## 保留元素

在幕后，Unstructured 为不同的文本块创建了不同的 "元素"。默认情况下，我们将它们组合在一起，但是您可以通过指定 `mode="elements"` 轻松保留这种分离。

```python
loader = UnstructuredMarkdownLoader(markdown_path, mode="elements")
data = loader.load()
print(f"文档数量：{len(data)}\n")
for document in data[:2]:
    print(f"{document}\n")
```

```output
文档数量：65
page_content='🦜️🔗 LangChain' metadata={'source': '../../../../README.md', 'last_modified': '2024-04-29T13:40:19', 'page_number': 1, 'languages': ['eng'], 'filetype': 'text/markdown', 'file_directory': '../../../..', 'filename': 'README.md', 'category': 'Title'}
page_content='⚡ Build context-aware reasoning applications ⚡' metadata={'source': '../../../../README.md', 'last_modified': '2024-04-29T13:40:19', 'page_number': 1, 'languages': ['eng'], 'parent_id': 'c3223b6f7100be08a78f1e8c0c28fde1', 'filetype': 'text/markdown', 'file_directory': '../../../..', 'filename': 'README.md', 'category': 'NarrativeText'}
```

请注意，在这种情况下，我们恢复了三种不同的元素类型：

```python
print(set(document.metadata["category"] for document in data))
```

```output
{'Title', 'NarrativeText', 'ListItem'}
```