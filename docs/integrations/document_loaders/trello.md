# Trello

[Trello](https://www.atlassian.com/software/trello) 是一个基于网络的项目管理和协作工具，允许个人和团队组织和跟踪他们的任务和项目。它提供了一个被称为“看板”的可视化界面，用户可以在其中创建列表和卡片来代表他们的任务和活动。

TrelloLoader 允许你从 Trello 看板加载卡片，并且是建立在 [py-trello](https://pypi.org/project/py-trello/) 之上的。

目前仅支持 `api_key/token`。

1. 凭证生成：https://trello.com/power-ups/admin/

2. 点击手动生成令牌链接以获取令牌。

要指定 API 密钥和令牌，你可以设置环境变量 ``TRELLO_API_KEY`` 和 ``TRELLO_TOKEN``，或者直接将 ``api_key`` 和 ``token`` 传递给 `from_credentials` 便捷构造方法。

该加载器允许你提供看板名称，以将相应的卡片加载到文档对象中。

请注意，官方文档中看板的“名称”也被称为“标题”：

https://support.atlassian.com/trello/docs/changing-a-boards-title-and-description/

你还可以指定几个加载参数，以包括/移除文档页面内容属性和元数据中的不同字段。

## 特点

- 从 Trello 看板加载卡片。

- 基于状态（打开或关闭）过滤卡片。

- 在加载的文档中包括卡片名称、评论和清单。

- 自定义要包含在文档中的额外元数据字段。

默认情况下，所有卡片字段都包括在完整文本页面内容和相应的元数据中。

```python
%pip install --upgrade --quiet  py-trello beautifulsoup4 lxml
```

```python
# 如果你已经使用环境变量设置了 API 密钥和令牌，
# 你可以跳过此单元格，并在下面的初始化步骤中注释掉 `api_key` 和 `token` 命名参数。
from getpass import getpass
API_KEY = getpass()
TOKEN = getpass()
```

```output
········
········
```

```python
from langchain_community.document_loaders import TrelloLoader
# 从“Awesome Board”获取打开的卡片
loader = TrelloLoader.from_credentials(
    "Awesome Board",
    api_key=API_KEY,
    token=TOKEN,
    card_filter="open",
)
documents = loader.load()
print(documents[0].page_content)
print(documents[0].metadata)
```

```output
Review Tech partner pages
Comments:
{'title': 'Review Tech partner pages', 'id': '6475357890dc8d17f73f2dcc', 'url': 'https://trello.com/c/b0OTZwkZ/1-review-tech-partner-pages', 'labels': ['Demand Marketing'], 'list': 'Done', 'closed': False, 'due_date': ''}
```

```python
# 从“Awesome Board”获取所有卡片，但只将卡片列表（列）包含为额外元数据。
loader = TrelloLoader.from_credentials(
    "Awesome Board",
    api_key=API_KEY,
    token=TOKEN,
    extra_metadata=("list"),
)
documents = loader.load()
print(documents[0].page_content)
print(documents[0].metadata)
```

```output
Review Tech partner pages
Comments:
{'title': 'Review Tech partner pages', 'id': '6475357890dc8d17f73f2dcc', 'url': 'https://trello.com/c/b0OTZwkZ/1-review-tech-partner-pages', 'list': 'Done'}
```

```python
# 从“Another Board”获取卡片，并从文档页面内容文本中排除卡片名称、清单和评论。
loader = TrelloLoader.from_credentials(
    "test",
    api_key=API_KEY,
    token=TOKEN,
    include_card_name=False,
    include_checklist=False,
    include_comments=False,
)
documents = loader.load()
print("Document: " + documents[0].page_content)
print(documents[0].metadata)
```