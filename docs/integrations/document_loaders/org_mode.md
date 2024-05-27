# Org-mode

[Org Mode](https://en.wikipedia.org/wiki/Org-mode) 是一种用于编辑、格式化和组织文档的模式，专为在自由软件文本编辑器 Emacs 中进行笔记、计划和撰写而设计。

## `UnstructuredOrgModeLoader`

您可以使用 `UnstructuredOrgModeLoader` 从 Org-mode 文件中加载数据，使用以下工作流程。

```python
from langchain_community.document_loaders import UnstructuredOrgModeLoader
```

```python
loader = UnstructuredOrgModeLoader(file_path="example_data/README.org", mode="elements")
docs = loader.load()
```

```python
print(docs[0])
```

```output
page_content='Example Docs' metadata={'source': 'example_data/README.org', 'filename': 'README.org', 'file_directory': 'example_data', 'filetype': 'text/org', 'page_number': 1, 'category': 'Title'}
```