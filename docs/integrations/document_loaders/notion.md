# Notion 数据库 1/2

>[Notion](https://www.notion.so/) 是一个协作平台，支持修改的 Markdown，并集成了看板、任务、维基和数据库功能。它是一个集合了笔记、知识和数据管理以及项目和任务管理的多合一工作空间。

本文介绍了如何从 Notion 数据库导入文档。

要获取 Notion 数据库导出文件，请按照以下步骤操作：

## 🧑 自定义数据集的导入说明

从 Notion 导出您的数据集。您可以点击右上角的三个点，然后点击“导出”来完成此操作。

在导出时，请确保选择“Markdown & CSV”格式选项。

这将在您的下载文件夹中生成一个 `.zip` 文件。将 `.zip` 文件移动到此存储库中。

运行以下命令解压缩 zip 文件（根据需要替换 `Export...` 为您自己的文件名）。

```shell
unzip Export-d3adfe0f-3131-4bf3-8987-a52017fc1bae.zip -d Notion_DB
```

运行以下命令导入数据。

```python
from langchain_community.document_loaders import NotionDirectoryLoader
```

```python
loader = NotionDirectoryLoader("Notion_DB")
```

```python
docs = loader.load()
```