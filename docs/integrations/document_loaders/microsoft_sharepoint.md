# 微软 SharePoint

[微软 SharePoint](https://en.wikipedia.org/wiki/SharePoint) 是一个基于网站的协作系统，使用工作流应用程序、“列表”数据库和其他网络部件以及安全功能，以赋予业务团队共同工作的能力，由微软开发。

本笔记涵盖了如何从[SharePoint文档库](https://support.microsoft.com/en-us/office/what-is-a-document-library-3b5976dd-65cf-4c9e-bf5a-713c10ca2872)加载文档。目前仅支持 docx、doc 和 pdf 文件。

## 先决条件

1. 使用[Microsoft身份平台](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)指南注册应用程序。

2. 注册完成后，Azure门户会显示应用注册的概述窗格。您会看到应用程序（客户端）ID。也称为 `client ID`，此值在微软身份平台中唯一标识您的应用程序。

3. 在您将要遵循的步骤中的 **item 1**，您可以将重定向 URI 设置为 `https://login.microsoftonline.com/common/oauth2/nativeclient`

4. 在您将要遵循的步骤中的 **item 1**，在“应用程序密码”部分生成一个新密码（`client_secret`）。

5. 按照此[文档](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-expose-web-apis#add-a-scope)的说明，向您的应用程序添加以下 `SCOPES`（`offline_access` 和 `Sites.Read.All`）。

6. 要从您的**文档库**中检索文件，您将需要其 ID。为了获得它，您将需要 `Tenant Name`、`Collection ID` 和 `Subsite ID` 的值。

7. 要找到您的 `Tenant Name`，请按照此[文档](https://learn.microsoft.com/en-us/azure/active-directory-b2c/tenant-management-read-tenant-name)的说明。一旦获得这个值，只需从该值中删除 `.onmicrosoft.com`，并将其余部分保留为您的 `Tenant Name`。

8. 要获取您的 `Collection ID` 和 `Subsite ID`，您将需要您的**SharePoint** `site-name`。您的 `SharePoint` 站点 URL 的格式如下 `https://<tenant-name>.sharepoint.com/sites/<site-name>`。此 URL 的最后一部分是 `site-name`。

9. 要获取站点 `Collection ID`，在浏览器中输入以下 URL：`https://<tenant>.sharepoint.com/sites/<site-name>/_api/site/id`，并复制 `Edm.Guid` 属性的值。

10. 要获取 `Subsite ID`（或 web ID），请使用：`https://<tenant>.sharepoint.com/sites/<site-name>/_api/web/id`，并复制 `Edm.Guid` 属性的值。

11. `SharePoint site ID` 的格式为：`<tenant-name>.sharepoint.com,<Collection ID>,<subsite ID>`。您可以保留该值以在下一步中使用。

12. 访问[Graph Explorer Playground](https://developer.microsoft.com/en-us/graph/graph-explorer)以获取您的 `Document Library ID`。第一步是确保您已使用与您的**SharePoint**站点相关联的帐户登录。然后，您需要向 `https://graph.microsoft.com/v1.0/sites/<SharePoint site ID>/drive` 发出请求，响应将返回一个包含字段 `id` 的有效负载，其中包含您的 `Document Library ID` 的 ID。

## 🧑 从SharePoint文档库摄取文档的说明

### 🔑 认证

默认情况下，`SharePointLoader` 期望 `CLIENT_ID` 和 `CLIENT_SECRET` 的值必须存储为名为 `O365_CLIENT_ID` 和 `O365_CLIENT_SECRET` 的环境变量。您可以通过应用程序根目录下的 `.env` 文件或在您的脚本中使用以下命令传递这些环境变量。

```python
os.environ['O365_CLIENT_ID'] = "YOUR CLIENT ID"
os.environ['O365_CLIENT_SECRET'] = "YOUR CLIENT SECRET"
```

此加载器使用一种称为[*代表用户*](https://learn.microsoft.com/en-us/graph/auth-v2-user?context=graph%2Fapi%2F1.0&view=graph-rest-1.0)的身份验证。这是一种需要用户同意的两步身份验证。当您实例化加载器时，它将调用打印一个 URL，用户必须访问该 URL以在所需权限上向应用程序授予同意。然后用户必须访问此 URL 并向应用程序授予同意。然后用户必须复制生成的页面 URL 并将其粘贴回控制台。如果登录尝试成功，该方法将返回 True。

```python
from langchain_community.document_loaders.sharepoint import SharePointLoader
loader = SharePointLoader(document_library_id="YOUR DOCUMENT LIBRARY ID")
```

一旦认证完成，加载器将在 `~/.credentials/` 文件夹中存储一个令牌（`o365_token.txt`）。稍后可以使用此令牌进行身份验证，而无需进行前面解释的复制/粘贴步骤。要在实例化加载器时使用此令牌进行身份验证，您需要将 `auth_with_token` 参数更改为 True。

```python
from langchain_community.document_loaders.sharepoint import SharePointLoader
loader = SharePointLoader(document_library_id="YOUR DOCUMENT LIBRARY ID", auth_with_token=True)
```

### 🗂️ 文档加载器

#### 📑 从文档库目录加载文档

`SharePointLoader` 可以从文档库的特定文件夹中加载文档。例如，您想要加载存储在文档库的 `Documents/marketing` 文件夹中的所有文档。

```python
from langchain_community.document_loaders.sharepoint import SharePointLoader
loader = SharePointLoader(document_library_id="您的文档库 ID", folder_path="Documents/marketing", auth_with_token=True)
documents = loader.load()
```

如果您收到 `Resource not found for the segment` 错误，请尝试使用 `folder_id` 而不是文件夹路径，可以从 [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/graph-explorer) 获取到。

```python
loader = SharePointLoader(document_library_id="您的文档库 ID", auth_with_token=True
                          folder_id="<folder-id>")
documents = loader.load()
```

如果您希望从根目录加载文档，可以省略 `folder_id`、`folder_path` 和 `documents_ids`，加载器将加载根目录。

```python
# 从根目录加载文档
loader = SharePointLoader(document_library_id="您的文档库 ID", auth_with_token=True)
documents = loader.load()
```

结合 `recursive=True`，您可以简单地从整个 SharePoint 加载所有文档：

```python
# 从根目录加载文档
loader = SharePointLoader(document_library_id="您的文档库 ID",
                          recursive=True,
                          auth_with_token=True)
documents = loader.load()
```

#### 📑 从文档 ID 列表加载文档

另一种可能性是提供要加载的每个文档的 `object_id` 列表。为此，您需要查询 [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/graph-explorer) 来查找您感兴趣的所有文档 ID。这个 [链接](https://learn.microsoft.com/en-us/graph/api/resources/onedrive?view=graph-rest-1.0#commonly-accessed-resources) 提供了一些有用的端点，可以帮助检索文档 ID。

例如，要检索存储在 `data/finance/` 文件夹中的所有对象的信息，您需要向以下地址发出请求：`https://graph.microsoft.com/v1.0/drives/<document-library-id>/root:/data/finance:/children`。一旦您获得了感兴趣的 ID 列表，然后您可以使用以下参数实例化加载器。

```python
from langchain_community.document_loaders.sharepoint import SharePointLoader
loader = SharePointLoader(document_library_id="您的文档库 ID", object_ids=["ID_1", "ID_2"], auth_with_token=True)
documents = loader.load()
```