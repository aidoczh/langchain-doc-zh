# Airbyte Shopify（已弃用）

注意：此特定连接器加载程序已被弃用。请改用 [`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于从 API、数据库和文件到数据仓库和数据湖的 ELT 管道的数据集成平台。它拥有最大的 ELT 连接器目录，可连接到数据仓库和数据库。

此加载程序将 Shopify 连接器公开为文档加载程序，允许您将各种 Shopify 对象加载为文档。

## 安装

首先，您需要安装 `airbyte-source-shopify` Python 包。

```python
%pip install --upgrade --quiet  airbyte-source-shopify
```

## 示例

查看 [Airbyte 文档页面](https://docs.airbyte.com/integrations/sources/shopify/) 以获取有关如何配置读取器的详细信息。配置对象应遵循的 JSON 模式可以在 Github 上找到：[https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-shopify/source_shopify/spec.json](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-shopify/source_shopify/spec.json)。

一般形式如下：

```python
{
    "start_date": "<要从中检索记录的日期的 ISO 格式，例如 2020-10-20T00:00:00Z>",
    "shop": "<要从中检索文档的商店名称>",
    "credentials": {
        "auth_method": "api_password",
        "api_password": "<您的 API 密码>"
    }
}
```

默认情况下，所有字段都存储为文档中的元数据，并且文本设置为空字符串。通过转换读取器返回的文档来构建文档的文本。

```python
from langchain_community.document_loaders.airbyte import AirbyteShopifyLoader
config = {
    # 您的 Shopify 配置
}
loader = AirbyteShopifyLoader(
    config=config, stream_name="orders"
)  # 请查看上面链接的文档以获取所有流的列表
```

现在，您可以按照通常的方式加载文档。

```python
docs = loader.load()
```

由于 `load` 返回一个列表，它将阻塞，直到加载所有文档。为了更好地控制这个过程，您还可以使用 `lazy_load` 方法，它返回一个迭代器。

```python
docs_iterator = loader.lazy_load()
```

请记住，默认情况下，页面内容为空，元数据对象包含记录的所有信息。要以不同的方式创建文档，请在创建加载程序时传入一个 record_handler 函数：

```python
from langchain_core.documents import Document
def handle_record(record, id):
    return Document(page_content=record.data["title"], metadata=record.data)
loader = AirbyteShopifyLoader(
    config=config, record_handler=handle_record, stream_name="orders"
)
docs = loader.load()
```

## 增量加载

一些流允许增量加载，这意味着源会跟踪已同步的记录，并且不会再次加载它们。这对于数据量大且经常更新的源非常有用。

要利用这一点，存储加载程序的 `last_state` 属性，并在再次创建加载程序时传入。这将确保只加载新记录。

```python
last_state = loader.last_state  # 安全存储
incremental_loader = AirbyteShopifyLoader(
    config=config, stream_name="orders", state=last_state
)
new_docs = incremental_loader.load()
```