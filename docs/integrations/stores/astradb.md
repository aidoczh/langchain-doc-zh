---

sidebar_label: Astra DB

---

# Astra DB

DataStax 的 [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 是建立在 Cassandra 上的无服务器向量数据库，通过易于使用的 JSON API 方便地提供。

`AstraDBStore` 和 `AstraDBByteStore` 需要安装 `astrapy` 包：

```python
%pip install --upgrade --quiet  astrapy
```

Store 需要以下参数：

- `api_endpoint`：Astra DB API 端点。看起来像 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`

- `token`：Astra DB 令牌。看起来像 `AstraCS:6gBhNmsk135....`

- `collection_name`：Astra DB 集合名称

- `namespace`：（可选）Astra DB 命名空间

## AstraDBStore

`AstraDBStore` 是 `BaseStore` 的实现，将所有内容存储在 DataStax Astra DB 实例中。

存储键必须是字符串，并将映射到 Astra DB 文档的 `_id` 字段。

存储值可以是任何可以由 `json.dumps` 序列化的对象。

在数据库中，条目的形式将是：

```json
{
  "_id": "<key>",
  "value": <value>
}
```

```python
from langchain_community.storage import AstraDBStore
```

```python
from getpass import getpass
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```

```python
store = AstraDBStore(
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    collection_name="my_store",
)
```

```python
store.mset([("k1", "v1"), ("k2", [0.1, 0.2, 0.3])])
print(store.mget(["k1", "k2"]))
```

```output
['v1', [0.1, 0.2, 0.3]]
```

### 与 CacheBackedEmbeddings 的使用

您可以将 `AstraDBStore` 与 [`CacheBackedEmbeddings`](/docs/how_to/caching_embeddings) 结合使用，以缓存嵌入计算的结果。

请注意，`AstraDBStore` 将嵌入存储为浮点数列表，而不是首先将它们转换为字节，因此我们不在那里使用 `fromByteStore`。

```python
from langchain.embeddings import CacheBackedEmbeddings
from langchain_openai import OpenAIEmbeddings
embeddings = CacheBackedEmbeddings(
    underlying_embeddings=OpenAIEmbeddings(), document_embedding_store=store
)
```

## AstraDBByteStore

`AstraDBByteStore` 是 `ByteStore` 的实现，将所有内容存储在 DataStax Astra DB 实例中。

存储键必须是字符串，并将映射到 Astra DB 文档的 `_id` 字段。

存储的 `bytes` 值将转换为 base64 字符串，以便存储到 Astra DB 中。

在数据库中，条目的形式将是：

```json
{
  "_id": "<key>",
  "value": "bytes encoded in base 64"
}
```

```python
from langchain_community.storage import AstraDBByteStore
```

```python
from getpass import getpass
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```

```python
store = AstraDBByteStore(
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    collection_name="my_store",
)
```

```python
store.mset([("k1", b"v1"), ("k2", b"v2")])
print(store.mget(["k1", "k2"]))
```

```output
[b'v1', b'v2']
```