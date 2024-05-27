# Notion 数据库 2/2

>[Notion](https://www.notion.so/) 是一个协作平台，支持修改后的 Markdown，并集成了看板、任务、维基和数据库。它是一个集合了笔记、知识和数据管理以及项目和任务管理的全能工作空间。

`NotionDBLoader` 是一个用于从 `Notion` 数据库加载内容的 Python 类。它从数据库中检索页面，读取其内容，并返回一个文档对象列表。

## 需求

- 一个 `Notion` 数据库

- Notion 集成令牌

## 设置

### 1. 创建 Notion 表格数据库

在 Notion 中创建一个新的表格数据库。您可以向数据库中添加任何列，并将它们视为元数据。例如，您可以添加以下列：

- 标题：将标题设置为默认属性。

- 类别：一个多选属性，用于存储与页面相关的类别。

- 关键词：一个多选属性，用于存储与页面相关的关键词。

将内容添加到数据库中每个页面的正文中。NotionDBLoader 将从这些页面中提取内容和元数据。

### 2. 创建 Notion 集成

要创建 Notion 集成，请按照以下步骤操作：

1. 访问 [Notion 开发者](https://www.notion.com/my-integrations) 页面，并使用您的 Notion 帐户登录。

2. 点击“+ 新建集成”按钮。

3. 给您的集成命名，并选择您的数据库所在的工作区。

4. 选择所需的功能，此扩展程序只需要“读取内容”功能。

5. 点击“提交”按钮以创建集成。

创建集成后，您将获得一个“集成令牌 (API 密钥)”。复制此令牌并妥善保管，因为您将需要它来使用 NotionDBLoader。

### 3. 将集成连接到数据库

要将您的集成连接到数据库，请按照以下步骤操作：

1. 在 Notion 中打开您的数据库。

2. 点击数据库视图右上角的三个点菜单图标。

3. 点击“+ 新建集成”按钮。

4. 找到您的集成，您可能需要在搜索框中开始输入其名称。

5. 点击“连接”按钮以将集成连接到数据库。

### 4. 获取数据库 ID

要获取数据库 ID，请按照以下步骤操作：

1. 在 Notion 中打开您的数据库。

2. 点击数据库视图右上角的三个点菜单图标。

3. 从菜单中选择“复制链接”以将数据库 URL 复制到剪贴板。

4. 数据库 ID 是在 URL 中找到的一长串字母数字字符。它通常看起来像这样：https://www.notion.so/username/8935f9d140a04f95a872520c4f123456?v=.... 在此示例中，数据库 ID 是 8935f9d140a04f95a872520c4f123456。

数据库设置完善，集成令牌和数据库 ID 已准备就绪，您现在可以使用 NotionDBLoader 代码从 Notion 数据库中加载内容和元数据。

## 使用

NotionDBLoader 是 langchain 包的文档加载器的一部分。您可以按以下方式使用它：

```python
from getpass import getpass
NOTION_TOKEN = getpass()
DATABASE_ID = getpass()
```

```output
········
········
```

```python
from langchain_community.document_loaders import NotionDBLoader
```

```python
loader = NotionDBLoader(
    integration_token=NOTION_TOKEN,
    database_id=DATABASE_ID,
    request_timeout_sec=30,  # 可选，默认为 10
)
```

```python
docs = loader.load()
```

```python
print(docs)
```

```output
```