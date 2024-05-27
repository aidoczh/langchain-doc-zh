# 微软 OneDrive

[微软 OneDrive](https://en.wikipedia.org/wiki/OneDrive)（原名 `SkyDrive`）是由微软运营的文件托管服务。

本文档介绍了如何从 `OneDrive` 加载文档。目前，仅支持 docx、doc 和 pdf 文件。

## 先决条件

1. 根据[Microsoft 身份平台](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)的说明注册一个应用程序。

2. 注册完成后，Azure 门户会显示应用注册的概述窗格。您会看到应用程序（客户端）ID。也称为 `client ID`，此值在 Microsoft 身份平台中唯一标识您的应用程序。

3. 在您将要执行的**第 1 项**步骤中，您可以将重定向 URI 设置为 `http://localhost:8000/callback`。

4. 在您将要执行的**第 1 项**步骤中，生成一个新密码（`client_secret`），位于“应用程序密码”部分。

5. 按照此[文档](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-expose-web-apis#add-a-scope)的说明，向您的应用程序添加以下 `SCOPES`（`offline_access` 和 `Files.Read.All`）。

6. 访问[Graph Explorer Playground](https://developer.microsoft.com/en-us/graph/graph-explorer)以获取您的 `OneDrive ID`。第一步是确保您已登录到与您的 OneDrive 帐户关联的帐户。然后，您需要向 `https://graph.microsoft.com/v1.0/me/drive` 发出请求，响应将返回一个包含字段 `id` 的有效负载，其中包含您的 OneDrive 帐户的 ID。

7. 您需要使用命令 `pip install o365` 安装 o365 包。

8. 完成这些步骤后，您必须拥有以下值：

- `CLIENT_ID`

- `CLIENT_SECRET`

- `DRIVE_ID`

## 🧑 从 OneDrive 获取文档的说明

### 🔑 身份验证

默认情况下，`OneDriveLoader` 期望 `CLIENT_ID` 和 `CLIENT_SECRET` 的值必须存储为名为 `O365_CLIENT_ID` 和 `O365_CLIENT_SECRET` 的环境变量。您可以通过应用程序根目录下的 `.env` 文件或在您的脚本中使用以下命令传递这些环境变量。

```python
os.environ['O365_CLIENT_ID'] = "YOUR CLIENT ID"
os.environ['O365_CLIENT_SECRET'] = "YOUR CLIENT SECRET"
```

此加载程序使用一种称为[*代表用户*](https://learn.microsoft.com/en-us/graph/auth-v2-user?context=graph%2Fapi%2F1.0&view=graph-rest-1.0)的身份验证。这是一种需要用户同意的两步身份验证。当您实例化加载程序时，它将调用打印一个 URL，用户必须访问该 URL 以在所需权限上同意该应用程序。然后用户必须访问此 URL 并同意该应用程序。然后用户必须复制生成的页面 URL 并将其粘贴回控制台。如果登录尝试成功，该方法将返回 True。

```python
from langchain_community.document_loaders.onedrive import OneDriveLoader
loader = OneDriveLoader(drive_id="YOUR DRIVE ID")
```

身份验证完成后，加载程序将在 `~/.credentials/` 文件夹中存储一个令牌（`o365_token.txt`）。稍后可以使用此令牌进行身份验证，而无需执行前面解释的复制/粘贴步骤。要在实例化加载程序时使用此令牌进行身份验证，您需要将 `auth_with_token` 参数更改为 True。

```python
from langchain_community.document_loaders.onedrive import OneDriveLoader
loader = OneDriveLoader(drive_id="YOUR DRIVE ID", auth_with_token=True)
```

### 🗂️ 文档加载程序

#### 📑 从 OneDrive 目录加载文档

`OneDriveLoader` 可以从您的 OneDrive 中的特定文件夹加载文档。例如，您想要加载存储在您的 OneDrive 中的 `Documents/clients` 文件夹中的所有文档。

```python
from langchain_community.document_loaders.onedrive import OneDriveLoader
loader = OneDriveLoader(drive_id="YOUR DRIVE ID", folder_path="Documents/clients", auth_with_token=True)
documents = loader.load()
```

#### 📑 从文档 ID 列表加载文档

另一种可能性是提供每个要加载的文档的 `object_id` 列表。为此，您需要查询[Microsoft Graph API](https://developer.microsoft.com/en-us/graph/graph-explorer)以查找您感兴趣的所有文档 ID。此[链接](https://learn.microsoft.com/en-us/graph/api/resources/onedrive?view=graph-rest-1.0#commonly-accessed-resources)提供了一组有助于检索文档 ID 的端点列表。

例如，要检索存储在“文档”文件夹根目录下的所有对象的信息，您需要向以下地址发出请求：`https://graph.microsoft.com/v1.0/drives/{YOUR DRIVE ID}/root/children`。一旦您获得了您感兴趣的ID列表，您可以使用以下参数实例化加载器。

```python
from langchain_community.document_loaders.onedrive import OneDriveLoader
loader = OneDriveLoader(drive_id="YOUR DRIVE ID", object_ids=["ID_1", "ID_2"], auth_with_token=True)
documents = loader.load()
```