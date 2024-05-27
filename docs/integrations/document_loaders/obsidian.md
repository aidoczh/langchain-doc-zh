# Obsidian

[Obsidian](https://obsidian.md/) 是一个功能强大且可扩展的知识库，它可以在本地的纯文本文件夹上运行。

本笔记本介绍了如何从 `Obsidian` 数据库加载文档。

由于 `Obsidian` 仅以 Markdown 文件夹的形式存储在磁盘上，因此加载程序只需获取到该目录的路径。

`Obsidian` 文件有时还包含[元数据](https://help.obsidian.md/Editing+and+formatting/Metadata)，这是文件顶部的 YAML 块。这些值将被添加到文档的元数据中。（`ObsidianLoader` 也可以传递 `collect_metadata=False` 参数来禁用此行为。）

```python
from langchain_community.document_loaders import ObsidianLoader
```

```python
loader = ObsidianLoader("<path-to-obsidian>")
```

```python
docs = loader.load()
```