# Quip

[Quip](https://quip.com) 是一款面向移动端和 Web 的协作办公软件套件。它允许一群人一起创建和编辑文档和电子表格，通常用于商务目的。

一个用于 `Quip` 文档的加载器。

请参考[这里](https://quip.com/dev/automation/documentation/current#section/Authentication/Get-Access-to-Quip's-APIs)了解如何获取个人访问令牌。

指定一个 `folder_ids` 和/或 `thread_ids` 列表，以加载相应文档到 Document 对象中，如果两者都被指定，加载器将基于 `folder_ids` 获取属于该文件夹的所有 `thread_ids`，并与传递的 `thread_ids` 结合，返回两个集合的并集。

* 如何获取 folder_id？

  进入 quip 文件夹，右键点击文件夹并复制链接，从链接中提取后缀作为 folder_id。提示：`https://example.quip.com/<folder_id>`

* 如何获取 thread_id？

  thread_id 即为文档 id。进入 quip 文档，右键点击文档并复制链接，从链接中提取后缀作为 thread_id。提示：`https://exmaple.quip.com/<thread_id>`

您还可以将 `include_all_folders` 设置为 `True`，以获取 group_folder_ids，并且您还可以指定一个布尔值 `include_attachments` 来包括附件，默认设置为 False，如果设置为 True，将下载所有附件，并且 QuipLoader 将从附件中提取文本并将其添加到 Document 对象中。目前支持的附件类型包括：`PDF`、`PNG`、`JPEG/JPG`、`SVG`、`Word` 和 `Excel`。此外，您可以指定一个布尔值 `include_comments` 来包括文档中的评论，默认设置为 False，如果设置为 True，将获取文档中的所有评论，并且 QuipLoader 将其添加到 Document 对象中。

在使用 QuipLoader 之前，请确保已安装 quip-api 包的最新版本：

```python
%pip install --upgrade --quiet  quip-api
```

## 示例

### 个人访问令牌

```python
from langchain_community.document_loaders.quip import QuipLoader
loader = QuipLoader(
    api_url="https://platform.quip.com", access_token="change_me", request_timeout=60
)
documents = loader.load(
    folder_ids={"123", "456"},
    thread_ids={"abc", "efg"},
    include_attachments=False,
    include_comments=False,
)
```