# 谷歌云盘
[谷歌云盘](https://en.wikipedia.org/wiki/Google_Drive) 是由谷歌开发的文件存储和同步服务。
本文介绍了如何从 `谷歌云盘` 加载文档。目前，仅支持加载 `谷歌文档`。
## 先决条件
1. 创建一个谷歌云项目或使用现有项目
2. 启用 [谷歌云盘 API](https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com)
3. [为桌面应用程序授权凭据](https://developers.google.com/drive/api/quickstart/python#authorize_credentials_for_a_desktop_application)
4. 运行 `pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib`
## 🧑 摄取谷歌文档数据的说明
将环境变量 `GOOGLE_APPLICATION_CREDENTIALS` 设置为空字符串 (`""`)。
默认情况下，`GoogleDriveLoader` 期望 `credentials.json` 文件位于 `~/.credentials/credentials.json`，但可以使用 `credentials_path` 关键字参数进行配置。`token.json` 也是一样，默认路径为 `~/.credentials/token.json`，构造函数参数为 `token_path`。
第一次使用 GoogleDriveLoader 时，您将在浏览器中看到用户认证的同意屏幕。认证后，`token.json` 将自动在提供的路径或默认路径创建。此外，如果该路径已经存在 `token.json`，则不会提示进行认证。
`GoogleDriveLoader` 可以从谷歌文档文档 id 列表或文件夹 id 加载。您可以从 URL 中获取文件夹和文档 id：
- 文件夹：https://drive.google.com/drive/u/0/folders/1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5 -> 文件夹 id 为 `"1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5"`
- 文档：https://docs.google.com/document/d/1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw/edit -> 文档 id 为 `"1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw"`
```python
%pip install --upgrade --quiet langchain-google-community[drive]
```
```python
from langchain_google_community import GoogleDriveLoader
```
```python
loader = GoogleDriveLoader(
    folder_id="1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5",
    token_path="/path/where/you/want/token/to/be/created/google_token.json",
    # 可选: 配置是否递归获取子文件夹中的文件。默认为 False。
    recursive=False,
)
```
```python
docs = loader.load()
```
当您传递一个 `folder_id` 时，默认加载所有类型为文档、表格和 PDF 的文件。您可以通过传递 `file_types` 参数修改此行为。
```python
loader = GoogleDriveLoader(
    folder_id="1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5",
    file_types=["document", "sheet"],
    recursive=False,
)
```
## 传入可选文件加载器
在处理除谷歌文档和谷歌表格之外的文件时，将可选文件加载器传递给 `GoogleDriveLoader` 可能会有所帮助。如果传递了文件加载器，那么该文件加载器将用于没有谷歌文档或谷歌表格 MIME 类型的文档。以下是使用文件加载器从谷歌云盘加载 Excel 文档的示例。
```python
from langchain_community.document_loaders import UnstructuredFileIOLoader
from langchain_google_community import GoogleDriveLoader
```
```python
file_id = "1x9WBtFPWMEAdjcJzPScRsjpjQvpSo_kz"
loader = GoogleDriveLoader(
    file_ids=[file_id],
    file_loader_cls=UnstructuredFileIOLoader,
    file_loader_kwargs={"mode": "elements"},
)
```
```python
docs = loader.load()
```
您还可以使用以下模式处理混合文件和谷歌文档/表格的文件夹。
```python
folder_id = "1asMOHY1BqBS84JcRbOag5LOJac74gpmD"
loader = GoogleDriveLoader(
    folder_id=folder_id,
    file_loader_cls=UnstructuredFileIOLoader,
    file_loader_kwargs={"mode": "elements"},
)
```
```python
docs = loader.load()
```
## 扩展用法
一个外部（非官方）组件可以管理谷歌云盘的复杂性：`langchain-googledrive`
它与 `langchain_community.document_loaders.GoogleDriveLoader` 兼容，并可以替代使用。
为了与容器兼容，认证使用环境变量 `GOOGLE_ACCOUNT_FILE` 作为凭据文件（用于用户或服务）。
```python
%pip install --upgrade --quiet  langchain-googledrive
```
```python
folder_id = "root"
# folder_id='1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5'
```
```python
# 使用高级版本。
from langchain_googledrive.document_loaders import GoogleDriveLoader
```
```python
loader = GoogleDriveLoader(
    folder_id=folder_id,
    recursive=False,
    num_results=2,  # 最大要加载的文件数
)
```
默认情况下，所有具有以下 MIME 类型的文件都可以转换为 `Document`。
- text/text
- text/plain
- text/html
- text/csv
- text/markdown [20]
- image/png
- image/jpeg
- application/epub+zip
- application/pdf
- application/rtf
- application/vnd.google-apps.document (GDoc)
- application/vnd.google-apps.presentation (GSlide)
- application/vnd.google-apps.spreadsheet (GSheet)
- application/vnd.google.colaboratory (Notebook colab)
- application/vnd.openxmlformats-officedocument.presentationml.presentation (PPTX)
- application/vnd.openxmlformats-officedocument.wordprocessingml.document (DOCX)
可以更新或自定义这些内容。请参阅 `GDriveLoader` 的文档。
但是，相应的软件包必须已安装。
```python
%pip install --upgrade --quiet  unstructured
```
```python
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```
### 加载认证身份
Google Drive Loader 摄取的每个文件的授权身份可以与每个文档的元数据一起加载。
```python
from langchain_google_community import GoogleDriveLoader
loader = GoogleDriveLoader(
    folder_id=folder_id,
    load_auth=True,
    # 可选: 配置是否加载每个文档的授权身份。
)
doc = loader.load()
```
您可以传递 `load_auth=True`，以将 Google Drive 文档访问身份添加到元数据中。
```python
doc[0].metadata
```
### 加载扩展元数据
还可以在每个文档的元数据中获取以下额外字段：
- full_path - Google Drive 中文件的完整路径。
- owner - 文件的所有者。
- size - 文件的大小。
```python
from langchain_google_community import GoogleDriveLoader
loader = GoogleDriveLoader(
    folder_id=folder_id,
    load_extended_matadata=True,
    # 可选: 配置是否加载每个文档的扩展元数据。
)
doc = loader.load()
```
您可以传递 `load_extended_matadata=True`，以将 Google Drive 文档的扩展详细信息添加到元数据中。
```python
doc[0].metadata
```
### 自定义搜索模式
可以设置与 Google [`list()`](https://developers.google.com/drive/api/v3/reference/files/list) API 兼容的所有参数。
要指定 Google 请求的新模式，可以使用 `PromptTemplate()`。
提示的变量可以在构造函数中使用 `kwargs` 设置。
提供了一些预格式化的请求（使用 `{query}`，`{folder_id}` 和/或 `{mime_type}`）：
您可以自定义选择文件的条件。提供了一组预定义的过滤器：
| 模板                               | 描述                                                           |
| -------------------------------------- | --------------------------------------------------------------------- |
| gdrive-all-in-folder                   | 从 `folder_id` 返回所有兼容的文件                        |
| gdrive-query                           | 在所有驱动器中搜索 `query`                                          |
| gdrive-by-name                         | 搜索名称为 `query` 的文件                                        |
| gdrive-query-in-folder                 | 在 `folder_id` 中搜索 `query`（如果 `recursive=true`，则包括子文件夹）  |
| gdrive-mime-type                       | 搜索特定的 `mime_type`                                         |
| gdrive-mime-type-in-folder             | 在 `folder_id` 中搜索特定的 `mime_type`                          |
| gdrive-query-with-mime-type            | 用特定的 `mime_type` 搜索 `query`                            |
| gdrive-query-with-mime-type-and-folder | 用特定的 `mime_type` 和 `folder_id` 搜索 `query`         |
```python
loader = GoogleDriveLoader(
    folder_id=folder_id,
    recursive=False,
    template="gdrive-query",  # 要使用的默认模板
    query="machine learning",
    num_results=2,  # 要加载的最大文件数
    supportsAllDrives=False,  # GDrive `list()` 参数
)
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```
您可以自定义您的模式。
```python
from langchain_core.prompts.prompt import PromptTemplate
loader = GoogleDriveLoader(
    folder_id=folder_id,
    recursive=False,
    template=PromptTemplate(
        input_variables=["query", "query_name"],
        template="fullText contains '{query}' and name contains '{query_name}' and trashed=false",
    ),  # 要使用的默认模板
    query="machine learning",
    query_name="ML",
    num_results=2,  # 要加载的最大文件数
)
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```
转换可以以 Markdown 格式进行管理：
- 项目符号
- 链接
- 表格
- 标题
将属性 `return_link` 设置为 `True` 以导出链接。
#### GSlide 和 GSheet 的模式
参数 mode 接受不同的值：
- "document"：返回每个文档的正文
- "snippets": 返回每个文件的描述（在 Google Drive 文件的元数据中设置）。
参数 `gslide_mode` 接受不同的值：
- "single"：一个包含&lt;PAGE BREAK&gt;的文档
- "slide"：每张幻灯片一个文档
- "elements"：每个元素一个文档
```python
loader = GoogleDriveLoader(
    template="gdrive-mime-type",
    mime_type="application/vnd.google-apps.presentation",  # 仅限 GSlide 文件
    gslide_mode="slide",
    num_results=2,  # 要加载的最大文件数
)
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```
参数 `gsheet_mode` 接受不同的值：
- `"single"`：按行生成一个文档
- `"elements"`：一个包含 Markdown 数组和&lt;PAGE BREAK&gt;标记的文档
```python
loader = GoogleDriveLoader(
    template="gdrive-mime-type",
    mime_type="application/vnd.google-apps.spreadsheet",  # 仅限 GSheet 文件
    gsheet_mode="elements",
    num_results=2,  # 要加载的最大文件数
)
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```
### 高级用法
所有 Google 文件都在元数据中有一个“描述”字段。此字段可用于记录文档摘要或其他索引标签（参见方法 `lazy_update_description_with_summary()`）。
如果使用 `mode="snippet"`，则仅使用描述作为正文。否则，`metadata['summary']` 字段可用。
有时，可以使用特定的过滤器从文件名中提取信息，以选择符合特定标准的文件。您可以使用过滤器。
有时会返回许多文档。不必同时将所有文档保存在内存中。您可以使用方法的延迟版本，逐个获取一个文档。最好使用复杂查询代替递归搜索。对于每个文件夹，如果启用 `recursive=True`，必须应用一个查询。
```python
import os
loader = GoogleDriveLoader(
    gdrive_api_file=os.environ["GOOGLE_ACCOUNT_FILE"],
    num_results=2,
    template="gdrive-query",
    filter=lambda search, file: "#test" not in file.get("description", ""),
    query="machine learning",
    supportsAllDrives=False,
)
for doc in loader.load():
    print("---")
    print(doc.page_content.strip()[:60] + "...")
```