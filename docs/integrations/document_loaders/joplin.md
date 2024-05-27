# Joplin

[Joplin](https://joplinapp.org/) 是一款开源的笔记应用。您可以记录自己的想法，并安全地从任何设备访问它们。

这本笔记本介绍了如何从 `Joplin` 数据库中加载文档。

`Joplin` 提供了用于访问其本地数据库的 [REST API](https://joplinapp.org/api/references/rest_api/)。此加载程序使用 API 来检索数据库中的所有笔记及其元数据。这需要一个访问令牌，可以通过以下步骤从应用程序中获取：

1. 打开 `Joplin` 应用。在加载文档时，应用程序必须保持打开状态。

2. 转到设置/选项，然后选择 "Web Clipper"。

3. 确保 Web Clipper 服务已启用。

4. 在 "高级选项" 下，复制授权令牌。

您可以直接使用访问令牌初始化加载程序，也可以将其存储在环境变量 JOPLIN_ACCESS_TOKEN 中。

除此之外，还有一种方法是将 `Joplin` 的笔记数据库导出为 Markdown 文件（可选地，包含 Front Matter 元数据），然后使用 Markdown 加载程序（例如 ObsidianLoader）来加载它们。

```python
from langchain_community.document_loaders import JoplinLoader
```

```python
loader = JoplinLoader(access_token="<access-token>")
```

```python
docs = loader.load()
```