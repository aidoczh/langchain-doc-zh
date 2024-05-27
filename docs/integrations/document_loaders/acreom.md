# acreom

[acreom](https://acreom.com) 是一个以开发者为先的知识库，任务运行在本地的 markdown 文件上。

以下是一个示例，演示如何将本地的 acreom 保险库加载到 Langchain 中。由于 acreom 中的本地保险库是一组纯文本的 .md 文件夹，加载程序需要指定目录的路径。

保险库文件可能包含一些存储为 YAML 头部的元数据。如果 `collect_metadata` 设置为 true，这些值将被添加到文档的元数据中。

```python
from langchain_community.document_loaders import AcreomLoader
```

```python
loader = AcreomLoader("<path-to-acreom-vault>", collect_metadata=False)
```

```python
docs = loader.load()
```