# Airbyte Hubspot（已弃用）

注意：`AirbyteHubspotLoader`已被弃用。请改用[`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte)是一个用于从API、数据库和文件到数据仓库和数据湖的ELT管道的数据集成平台。它拥有最大的ELT连接器目录，可连接到数据仓库和数据库。

该加载器将Hubspot连接器公开为文档加载器，允许您将各种Hubspot对象加载为文档。

## 安装

首先，您需要安装`airbyte-source-hubspot` Python包。

```python
%pip install --upgrade --quiet  airbyte-source-hubspot
```

## 示例

查看[Airbyte文档页面](https://docs.airbyte.com/integrations/sources/hubspot/)，了解如何配置读取器的详细信息。配置对象应遵循的JSON模式可以在Github上找到：[https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-hubspot/source_hubspot/spec.yaml](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-hubspot/source_hubspot/spec.yaml)。

一般形式如下：

```python
{
  "start_date": "<要从中检索记录的日期的ISO格式，例如2020-10-20T00:00:00Z>",
  "credentials": {
    "credentials_title": "私人应用凭证",
    "access_token": "<您的私人应用访问令牌>"
  }
}
```

默认情况下，所有字段都存储为文档中的元数据，文本设置为空字符串。通过转换读取器返回的文档来构建文档的文本。

```python
from langchain_community.document_loaders.airbyte import AirbyteHubspotLoader
config = {
    # 您的Hubspot配置
}
loader = AirbyteHubspotLoader(
    config=config, stream_name="products"
)  # 查看上面链接的文档以获取所有流的列表
```

现在您可以按照通常的方式加载文档。

```python
docs = loader.load()
```

由于`load`返回一个列表，它将阻塞直到所有文档加载完毕。为了更好地控制这个过程，您也可以使用`lazy_load`方法，它返回一个迭代器：

```python
docs_iterator = loader.lazy_load()
```

请注意，默认情况下，页面内容为空，元数据对象包含记录的所有信息。要处理文档，创建一个从基本加载器继承并自己实现`_handle_records`方法的类：

```python
from langchain_core.documents import Document
def handle_record(record, id):
    return Document(page_content=record.data["title"], metadata=record.data)
loader = AirbyteHubspotLoader(
    config=config, record_handler=handle_record, stream_name="products"
)
docs = loader.load()
```

## 增量加载

一些流支持增量加载，这意味着源会跟踪已同步的记录，不会再次加载它们。这对于数据量大且经常更新的源非常有用。

要利用这一点，存储加载器的`last_state`属性，并在再次创建加载器时传递它。这将确保只加载新记录。

```python
last_state = loader.last_state  # 安全存储
incremental_loader = AirbyteHubspotLoader(
    config=config, stream_name="products", state=last_state
)
new_docs = incremental_loader.load()
```