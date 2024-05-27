# 微软 OneNote

本笔记本涵盖了如何从 `OneNote` 中加载文档。

## 先决条件

1. 按照[Microsoft 身份平台](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)的说明注册一个应用程序。

2. 注册完成后，Azure 门户会显示应用注册的概述窗格。您会看到应用程序 (客户端) ID。也称为 `客户端 ID`，此值在 Microsoft 身份平台中唯一标识您的应用程序。

3. 在您将要遵循的步骤中的 **项目 1**，您可以将重定向 URI 设置为 `http://localhost:8000/callback`

4. 在您将要遵循的步骤中的 **项目 1**，在“应用程序密码”部分生成一个新密码 (`client_secret`)。

5. 按照此[文档](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-expose-web-apis#add-a-scope)的说明，向您的应用程序添加以下 `SCOPES` (`Notes.Read`)。

6. 您需要使用以下命令安装 msal 和 bs4 包：`pip install msal` 和 `pip install beautifulsoup4`。

7. 完成这些步骤后，您必须拥有以下值：

- `CLIENT_ID`

- `CLIENT_SECRET`

## 🧑 从 OneNote 中摄取文档的说明

### 🔑 认证

默认情况下，`OneNoteLoader` 期望 `CLIENT_ID` 和 `CLIENT_SECRET` 的值必须存储为名为 `MS_GRAPH_CLIENT_ID` 和 `MS_GRAPH_CLIENT_SECRET` 的环境变量。您可以通过应用程序根目录下的 `.env` 文件或在您的脚本中使用以下命令传递这些环境变量。

```python
os.environ['MS_GRAPH_CLIENT_ID'] = "YOUR CLIENT ID"
os.environ['MS_GRAPH_CLIENT_SECRET'] = "YOUR CLIENT SECRET"
```

此加载器使用一种称为[*代表用户*](https://learn.microsoft.com/en-us/graph/auth-v2-user?context=graph%2Fapi%2F1.0&view=graph-rest-1.0)的认证。这是一种需要用户同意的两步认证。当您实例化加载器时，它将调用打印一个 URL，用户必须访问该 URL 同意应用程序的所需权限。然后用户必须访问此 URL 并同意该应用程序。然后用户必须复制生成的页面 URL 并将其粘贴回控制台。如果登录尝试成功，该方法将返回 True。

```python
from langchain_community.document_loaders.onenote import OneNoteLoader
loader = OneNoteLoader(notebook_name="NOTEBOOK NAME", section_name="SECTION NAME", page_title="PAGE TITLE")
```

认证完成后，加载器将在 `~/.credentials/` 文件夹中存储一个令牌 (`onenote_graph_token.txt`)。稍后可以使用此令牌进行身份验证，而无需进行前面解释的复制/粘贴步骤。要在加载器实例化中将 `auth_with_token` 参数更改为 True 以使用此令牌进行身份验证。

```python
from langchain_community.document_loaders.onenote import OneNoteLoader
loader = OneNoteLoader(notebook_name="NOTEBOOK NAME", section_name="SECTION NAME", page_title="PAGE TITLE", auth_with_token=True)
```

或者，您还可以直接将令牌传递给加载器。当您希望使用另一个应用程序生成的令牌进行身份验证时，这将非常有用。例如，您可以使用[Microsoft Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)生成一个令牌，然后将其传递给加载器。

```python
from langchain_community.document_loaders.onenote import OneNoteLoader
loader = OneNoteLoader(notebook_name="NOTEBOOK NAME", section_name="SECTION NAME", page_title="PAGE TITLE", access_token="TOKEN")
```

### 🗂️ 文档加载器

#### 📑 从 OneNote 笔记本加载页面

`OneNoteLoader` 可以从存储在 OneDrive 中的 OneNote 笔记本中加载页面。您可以指定任何组合的 `notebook_name`、`section_name`、`page_title` 来过滤特定笔记本下的页面、特定部分下的页面或具有特定标题的页面。例如，您想要加载存储在名为 `Recipes` 的部分下的所有页面。

```python
from langchain_community.document_loaders.onenote import OneNoteLoader
loader = OneNoteLoader(section_name="Recipes", auth_with_token=True)
documents = loader.load()
```

#### 📑 从页面 ID 列表加载页面

另一种可能性是提供每个要加载的页面的 `object_ids` 列表。为此，您需要查询[Microsoft Graph API](https://developer.microsoft.com/en-us/graph/graph-explorer)以查找您感兴趣的所有文档 ID。此[链接](https://learn.microsoft.com/en-us/graph/onenote-get-content#page-collection)提供了一组有助于检索文档 ID 的端点列表。

例如，要检索存储在您的笔记本中的所有页面信息，您需要向以下地址发出请求：`https://graph.microsoft.com/v1.0/me/onenote/pages`。一旦您获得了感兴趣的页面ID列表，然后您可以使用以下参数实例化加载器。

```python
from langchain_community.document_loaders.onenote import OneNoteLoader
loader = OneNoteLoader(object_ids=["ID_1", "ID_2"], auth_with_token=True)
documents = loader.load()
```