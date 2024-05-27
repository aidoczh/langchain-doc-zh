# 缓存

嵌入可以被存储或临时缓存，以避免需要重新计算它们。

可以使用 `CacheBackedEmbeddings` 来缓存嵌入。缓存支持的嵌入器是一个包装器，它在键值存储中缓存嵌入。文本被哈希处理，哈希值被用作缓存中的键。

初始化 `CacheBackedEmbeddings` 的主要支持方式是 `from_bytes_store`。它接受以下参数：

- underlying_embedder: 用于嵌入的嵌入器。

- document_embedding_cache: 用于缓存文档嵌入的任何 [`ByteStore`](/docs/integrations/stores/)。

- batch_size: (可选，默认为 `None`) 在存储更新之间要嵌入的文档数量。

- namespace: (可选，默认为 `""`) 用于文档缓存的命名空间。此命名空间用于避免与其他缓存发生冲突。例如，将其设置为所使用的嵌入模型的名称。

- query_embedding_cache: (可选，默认为 `None` 或不缓存) 用于缓存查询嵌入的 [`ByteStore`](/docs/integrations/stores/)，或者设置为 `True` 以使用与 `document_embedding_cache` 相同的存储。

**注意**：

- 请务必设置 `namespace` 参数，以避免使用不同嵌入模型嵌入的相同文本发生冲突。

- `CacheBackedEmbeddings` 默认不缓存查询嵌入。要启用查询缓存，需要指定一个 `query_embedding_cache`。

```python
from langchain.embeddings import CacheBackedEmbeddings
```

## 使用向量存储

首先，让我们看一个示例，该示例使用本地文件系统存储嵌入，并使用 FAISS 向量存储进行检索。

```python
%pip install --upgrade --quiet  langchain-openai faiss-cpu
```

```python
from langchain.storage import LocalFileStore
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
underlying_embeddings = OpenAIEmbeddings()
store = LocalFileStore("./cache/")
cached_embedder = CacheBackedEmbeddings.from_bytes_store(
    underlying_embeddings, store, namespace=underlying_embeddings.model
)
```

在嵌入之前，缓存是空的：

```python
list(store.yield_keys())
```

```output
[]
```

加载文档，将其分割成块，嵌入每个块并将其加载到向量存储中。

```python
raw_documents = TextLoader("state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
documents = text_splitter.split_documents(raw_documents)
```

创建向量存储：

```python
%%time
db = FAISS.from_documents(documents, cached_embedder)
```

```output
CPU times: user 218 ms, sys: 29.7 ms, total: 248 ms
Wall time: 1.02 s
```

如果我们尝试再次创建向量存储，速度会快得多，因为不需要重新计算任何嵌入。

```python
%%time
db2 = FAISS.from_documents(documents, cached_embedder)
```

```output
CPU times: user 15.7 ms, sys: 2.22 ms, total: 18 ms
Wall time: 17.2 ms
```

这里是一些已创建的嵌入示例：

```python
list(store.yield_keys())[:5]
```

```output
['text-embedding-ada-00217a6727d-8916-54eb-b196-ec9c9d6ca472',
 'text-embedding-ada-0025fc0d904-bd80-52da-95c9-441015bfb438',
 'text-embedding-ada-002e4ad20ef-dfaa-5916-9459-f90c6d8e8159',
 'text-embedding-ada-002ed199159-c1cd-5597-9757-f80498e8f17b',
 'text-embedding-ada-0021297d37a-2bc1-5e19-bf13-6c950f075062']
```

# 更换 `ByteStore`

为了使用不同的 `ByteStore`，只需在创建 `CacheBackedEmbeddings` 时使用它。下面，我们创建一个等效的缓存嵌入对象，只是使用非持久的 `InMemoryByteStore`：

```python
from langchain.embeddings import CacheBackedEmbeddings
from langchain.storage import InMemoryByteStore
store = InMemoryByteStore()
cached_embedder = CacheBackedEmbeddings.from_bytes_store(
    underlying_embeddings, store, namespace=underlying_embeddings.model
)
```