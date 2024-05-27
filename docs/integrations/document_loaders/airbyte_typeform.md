# Airbyte Typeform（已弃用）

注意：此特定于连接器的加载程序已被弃用。请改用 [`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于从 API、数据库和文件到数据仓库和数据湖的 ELT 管道的数据集成平台。它拥有最大的 ELT 连接器目录，可连接到数据仓库和数据库。

此加载程序将 Typeform 连接器公开为文档加载程序，允许您将各种 Typeform 对象加载为文档。

## 安装

首先，您需要安装 `airbyte-source-typeform` Python 包。

```python
%pip install --upgrade --quiet  airbyte-source-typeform
```

## 示例

查看 [Airbyte 文档页面](https://docs.airbyte.com/integrations/sources/typeform/) 以获取有关如何配置读取器的详细信息。

配置对象应遵循的 JSON 模式可以在 Github 上找到：[https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-typeform/source_typeform/spec.json](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-typeform/source_typeform/spec.json)。

一般形式如下：

```python
{
  "credentials": {
    "auth_type": "Private Token",
    "access_token": "<your auth token>"
  },
  "start_date": "<date from which to start retrieving records from in ISO format, e.g. 2020-10-20T00:00:00Z>",
  "form_ids": ["<id of form to load records for>"] # if omitted, records from all forms will be loaded
}
```

默认情况下，所有字段都存储为文档中的元数据，并且文本设置为空字符串。通过转换读取器返回的文档来构建文档的文本。

```python
from langchain_community.document_loaders.airbyte import AirbyteTypeformLoader
config = {
    # 您的 Typeform 配置
}
loader = AirbyteTypeformLoader(
    config=config, stream_name="forms"
)  # 请查看上面链接的文档以获取所有流的列表
```

现在您可以按照通常的方式加载文档

```python
docs = loader.load()
```

由于 `load` 返回一个列表，它将阻塞，直到所有文档加载完成。为了更好地控制这个过程，您也可以使用 `lazy_load` 方法，它返回一个迭代器：

```python
docs_iterator = loader.lazy_load()
```

请记住，默认情况下页面内容为空，元数据对象包含记录的所有信息。要以不同的方式创建文档，请在创建加载程序时传入一个 record_handler 函数：

```python
from langchain_core.documents import Document
def handle_record(record, id):
    return Document(page_content=record.data["title"], metadata=record.data)
loader = AirbyteTypeformLoader(
    config=config, record_handler=handle_record, stream_name="forms"
)
docs = loader.load()
```

## 增量加载

一些流允许增量加载，这意味着源会跟踪已同步的记录，并且不会再次加载它们。这对于数据量大且经常更新的源非常有用。

要利用这一点，存储加载程序的 `last_state` 属性，并在再次创建加载程序时传入。这将确保只加载新记录。

```python
last_state = loader.last_state  # 安全存储
incremental_loader = AirbyteTypeformLoader(
    config=config, record_handler=handle_record, stream_name="forms", state=last_state
)
new_docs = incremental_loader.load()
```