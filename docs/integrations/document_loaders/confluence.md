# Confluence

>[Confluence](https://www.atlassian.com/software/confluence) 是一个维基协作平台，用于保存和组织与项目相关的所有材料。`Confluence` 是一个主要处理内容管理活动的知识库。

一个用于 `Confluence` 页面的加载器。

目前支持 `username/api_key`，`Oauth2 login`。此外，本地安装还支持 `token` 认证。

指定一个 `page_id` 列表和/或 `space_key`，将对应的页面加载到文档对象中，如果两者都指定了，将返回两个集合的并集。

您还可以指定一个布尔值 `include_attachments` 来包含附件，默认设置为 False，如果设置为 True，则会下载所有附件，并且 ConfluenceReader 将从附件中提取文本并将其添加到文档对象中。目前支持的附件类型有：`PDF`，`PNG`，`JPEG/JPG`，`SVG`，`Word` 和 `Excel`。

提示：`space_key` 和 `page_id` 都可以在 Confluence 页面的 URL 中找到 - https://yoursite.atlassian.com/wiki/spaces/<space_key>/pages/<page_id>

在使用 ConfluenceLoader 之前，请确保您已安装了最新版本的 atlassian-python-api 包：

```python
%pip install --upgrade --quiet  atlassian-python-api
```

## 示例

### 用户名和密码或用户名和 API 令牌（仅适用于 Atlassian Cloud）

此示例使用用户名和密码进行身份验证，或者如果您连接到托管在 Atlassian Cloud 上的 Confluence 版本，则使用用户名和 API 令牌进行身份验证。

您可以在此处生成 API 令牌：https://id.atlassian.com/manage-profile/security/api-tokens。

`limit` 参数指定在单个调用中将检索多少个文档，而不是总共将检索多少个文档。

默认情况下，代码将以 50 个文档批次返回最多 1000 个文档。要控制文档的总数，请使用 `max_pages` 参数。

请注意，atlassian-python-api 包中 `limit` 参数的最大值目前为 100。

```python
from langchain_community.document_loaders import ConfluenceLoader
loader = ConfluenceLoader(
    url="https://yoursite.atlassian.com/wiki", username="me", api_key="12345"
)
documents = loader.load(space_key="SPACE", include_attachments=True, limit=50)
```

### 个人访问令牌（仅适用于服务器/本地版）

此方法仅适用于数据中心/服务器本地版。

有关如何生成个人访问令牌（PAT）的更多信息，请查看官方 Confluence 文档：https://confluence.atlassian.com/enterprise/using-personal-access-tokens-1026032365.html。

使用 PAT 时，只需提供令牌值，无需提供用户名。

请注意，ConfluenceLoader 将在生成 PAT 的用户的权限下运行，并且只能加载该用户具有访问权限的文档。

```python
from langchain_community.document_loaders import ConfluenceLoader
loader = ConfluenceLoader(url="https://yoursite.atlassian.com/wiki", token="12345")
documents = loader.load(
    space_key="SPACE", include_attachments=True, limit=50, max_pages=50
)
```