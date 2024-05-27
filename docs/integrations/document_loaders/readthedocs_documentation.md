# ReadTheDocs 文档

>[Read the Docs](https://readthedocs.org/) 是一个开源的免费软件文档托管平台。它使用 `Sphinx` 文档生成器生成文档。

本文档介绍了如何加载作为 `Read-The-Docs` 构建的一部分生成的 HTML 内容。

例如，可以参考[这里](https://github.com/langchain-ai/chat-langchain)。

这假设 HTML 已经被抓取到一个文件夹中。可以通过取消注释并运行以下命令来完成此操作

```python
%pip install --upgrade --quiet  beautifulsoup4
```

```python
#!wget -r -A.html -P rtdocs https://python.langchain.com/en/latest/
```

```python
from langchain_community.document_loaders import ReadTheDocsLoader
```

```python
loader = ReadTheDocsLoader("rtdocs", features="html.parser")
```

```python
docs = loader.load()
```