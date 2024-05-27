# Airbyte CDK (已弃用)

注意：`AirbyteCDKLoader` 已弃用。请使用 [`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于从 API、数据库和文件到数据仓库和数据湖的 ELT 管道的数据集成平台。它拥有最大的 ELT 连接器目录，可连接到数据仓库和数据库。

许多源连接器是使用 [Airbyte CDK](https://docs.airbyte.com/connector-development/cdk-python/) 实现的。此加载器允许运行任何这些连接器并将数据返回为文档。

## 安装

首先，您需要安装 `airbyte-cdk` python 包。

```python
%pip install --upgrade --quiet  airbyte-cdk
```

然后，要么从 [Airbyte Github 仓库](https://github.com/airbytehq/airbyte/tree/master/airbyte-integrations/connectors) 安装现有连接器，要么使用 [Airbyte CDK](https://docs.airbyte.io/connector-development/connector-development) 创建自己的连接器。

例如，要安装 Github 连接器，请运行

```python
%pip install --upgrade --quiet  "source_github@git+https://github.com/airbytehq/airbyte.git@master#subdirectory=airbyte-integrations/connectors/source-github"
```

一些源也作为常规包发布在 PyPI 上。

## 示例

现在，您可以基于导入的源创建一个基于 `AirbyteCDKLoader` 的加载器。它接受一个传递给连接器的 `config` 对象。您还必须通过名称 (`stream_name`) 选择要检索记录的流。查看连接器的文档页面和规范定义，以获取有关配置对象和可用流的更多信息。对于 Github 连接器，这些是：

* [https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-github/source_github/spec.json](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-github/source_github/spec.json).

* [https://docs.airbyte.com/integrations/sources/github/](https://docs.airbyte.com/integrations/sources/github/)

```python
from langchain_community.document_loaders.airbyte import AirbyteCDKLoader
from source_github.source import SourceGithub  # 在此处插入您自己的源
config = {
    # 您的 Github 配置
    "credentials": {"api_url": "api.github.com", "personal_access_token": "<token>"},
    "repository": "<repo>",
    "start_date": "<date from which to start retrieving records from in ISO format, e.g. 2020-10-20T00:00:00Z>",
}
issues_loader = AirbyteCDKLoader(
    source_class=SourceGithub, config=config, stream_name="issues"
)
```

现在，您可以以通常的方式加载文档

```python
docs = issues_loader.load()
```

由于 `load` 返回一个列表，它将阻塞，直到所有文档加载完成。为了更好地控制此过程，您还可以使用 `lazy_load` 方法，该方法返回一个迭代器：

```python
docs_iterator = issues_loader.lazy_load()
```

请注意，默认情况下，页面内容为空，元数据对象包含记录的所有信息。要以不同的方式创建文档，请在创建加载器时传入 `record_handler` 函数：

```python
from langchain_core.documents import Document
def handle_record(record, id):
    return Document(
        page_content=record.data["title"] + "\n" + (record.data["body"] or ""),
        metadata=record.data,
    )
issues_loader = AirbyteCDKLoader(
    source_class=SourceGithub,
    config=config,
    stream_name="issues",
    record_handler=handle_record,
)
docs = issues_loader.load()
```

## 增量加载

一些流允许增量加载，这意味着源会跟踪已同步的记录，并且不会再次加载它们。这对于数据量大且经常更新的源非常有用。

要利用这一点，存储加载器的 `last_state` 属性，并在再次创建加载器时传入。这将确保只加载新记录。

```python
last_state = issues_loader.last_state  # 安全存储
incremental_issue_loader = AirbyteCDKLoader(
    source_class=SourceGithub, config=config, stream_name="issues", state=last_state
)
new_docs = incremental_issue_loader.load()
```