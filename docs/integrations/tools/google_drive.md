# 谷歌云端硬盘

本笔记本介绍了如何将 LangChain 连接到 `谷歌云端硬盘 API`。

## 先决条件

1. 创建一个谷歌云项目或使用现有项目

1. 启用 [谷歌云端硬盘 API](https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com)

1. [为桌面应用程序授权凭据](https://developers.google.com/drive/api/quickstart/python#authorize_credentials_for_a_desktop_application)

1. 运行 `pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib`

## 获取谷歌文档数据的说明

默认情况下，`GoogleDriveTools` 和 `GoogleDriveWrapper` 预期 `credentials.json` 文件位于 `~/.credentials/credentials.json`，但可以使用 `GOOGLE_ACCOUNT_FILE` 环境变量进行配置。`token.json` 的位置使用相同的目录（或使用参数 `token_path`）。请注意，第一次使用该工具时，`token.json` 将会自动创建。

`GoogleDriveSearchTool` 可以通过一些请求检索文件的选择。

默认情况下，如果您使用 `folder_id`，则可以将此文件夹中的所有文件检索到 `Document`，如果名称与查询匹配。

```python
%pip install --upgrade --quiet  google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

您可以从以下 URL 获取文件夹和文档 ID：

* 文件夹：https://drive.google.com/drive/u/0/folders/1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5 -> 文件夹 ID 为 `"1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5"`

* 文档：https://docs.google.com/document/d/1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw/edit -> 文档 ID 为 `"1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw"`

特殊值 `root` 代表您的个人主目录。

```python
folder_id = "root"
# folder_id='1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5'
```

默认情况下，所有具有以下 MIME 类型的文件都可以转换为 `Document`。

- text/text

- text/plain

- text/html

- text/csv

- text/markdown

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

可以更新或自定义这些设置。请参阅 `GoogleDriveAPIWrapper` 的文档。

但是，相应的软件包必须已安装。

```python
%pip install --upgrade --quiet  unstructured
```

```python
from langchain_googldrive.tools.google_drive.tool import GoogleDriveSearchTool
from langchain_googledrive.utilities.google_drive import GoogleDriveAPIWrapper
# 默认情况下，仅在文件名中搜索。
tool = GoogleDriveSearchTool(
    api_wrapper=GoogleDriveAPIWrapper(
        folder_id=folder_id,
        num_results=2,
        template="gdrive-query-in-folder",  # 在文档正文中搜索
    )
)
```

```python
import logging
logging.basicConfig(level=logging.INFO)
```

```python
tool.run("machine learning")
```

```python
tool.description
```

```python
from langchain.agents import load_tools
tools = load_tools(
    ["google-drive-search"],
    folder_id=folder_id,
    template="gdrive-query-in-folder",
)
```

## 在代理程序中使用

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
)
```

```python
agent.run("在谷歌云端硬盘中搜索，谁是 'Yann LeCun'？")
```