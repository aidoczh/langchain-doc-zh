# 如何使用 LangChain 索引 API
在这里，我们将介绍使用 LangChain 索引 API 进行基本索引工作流程。
索引 API 允许您从任何来源加载并同步文档到向量存储中。具体来说，它有助于：
- 避免将重复的内容写入向量存储
- 避免重写未更改的内容
- 避免在未更改的内容上重新计算嵌入
所有这些都应该为您节省时间和金钱，并改善您的向量搜索结果。
重要的是，索引 API 甚至可以处理已经经历了多个转换步骤（例如通过文本分块）的文档，而不是原始源文档。
## 工作原理
LangChain 索引利用记录管理器（`RecordManager`）来跟踪文档写入向量存储。
在索引内容时，为每个文档计算哈希，并将以下信息存储在记录管理器中：
- 文档哈希（页面内容和元数据的哈希）
- 写入时间
- 源 ID —— 每个文档的元数据应包含信息，以便我们确定此文档的最终来源
## 删除模式
将文档索引到向量存储时，可能需要删除向量存储中的一些现有文档。在某些情况下，您可能希望删除与正在索引的新文档来自相同来源的任何现有文档。在其他情况下，您可能希望彻底删除所有现有文档。索引 API 的删除模式让您选择所需的行为：
| 清理模式   | 去重内容 | 可并行化 | 清理已删除的源文档 | 清理源文档和/或派生文档的变异 | 清理时间   |
|------------|-----------|-----------|---------------------------|----------------------------------------|-------------|
| 无         | ✅        | ✅        | ❌                        | ❌                                     | -           |
| 增量       | ✅        | ✅        | ❌                        | ✅                                     | 持续       |
| 完全       | ✅        | ❌        | ✅                        | ✅                                     | 索引结束   |
`无` 不执行任何自动清理，允许用户手动清理旧内容。
`增量` 和 `完全` 提供以下自动清理功能：
- 如果源文档或派生文档的内容**发生更改**，`增量` 或 `完全` 模式将清理（删除）先前版本的内容。
- 如果源文档已被**删除**（意味着它不包括在当前正在索引的文档中），`完全` 清理模式将正确地从向量存储中删除它，但 `增量` 模式不会。
当内容发生变异（例如，源 PDF 文件已经修订）时，在索引期间会有一段时间，新旧版本都可能返回给用户。这发生在新内容被写入之后，但旧版本被删除之前。
- `增量` 索引最小化了这段时间，因为它能够连续进行清理，随着写入的进行。
- `完全` 模式在所有批次写入后进行清理。
## 要求
1. 不要与已经独立于索引 API 预先填充内容的存储一起使用，因为记录管理器将不知道先前已插入记录。
2. 仅适用于支持 LangChain `vectorstore` 的向量存储：
   - 通过 ID 添加文档（使用 `ids` 参数的 `add_documents` 方法）
   - 通过 ID 删除文档（使用 `ids` 参数的 `delete` 方法）
兼容的向量存储：`Aerospike`、`AnalyticDB`、`AstraDB`、`AwaDB`、`Bagel`、`Cassandra`、`Chroma`、`CouchbaseVectorStore`、`DashVector`、`DatabricksVectorSearch`、`DeepLake`、`Dingo`、`ElasticVectorSearch`、`ElasticsearchStore`、`FAISS`、`HanaDB`、`Milvus`、`MyScale`、`OpenSearchVectorSearch`、`PGVector`、`Pinecone`、`Qdrant`、`Redis`、`Rockset`、`ScaNN`、`SupabaseVectorStore`、`SurrealDBStore`、`TimescaleVector`、`Vald`、`VDMS`、`Vearch`、`VespaStore`、`Weaviate`、`ZepVectorStore`、`TencentVectorDB`、`OpenSearchVectorSearch`。
## 注意
记录管理器依赖于基于时间的机制来确定可以清理哪些内容（当使用 `完全` 或 `增量` 清理模式时）。
如果两个任务连续运行，并且第一个任务在时钟时间更改之前完成，那么第二个任务可能无法清理内容。
在实际设置中，出现这种情况的可能性不大，原因如下：
1. RecordManager 使用更高分辨率的时间戳。
2. 如果两个任务之间的时间间隔很短，数据在第一次和第二次任务运行之间需要发生变化，这种情况变得不太可能发生。
3. 索引任务通常需要超过几毫秒的时间。
## 快速入门
```python
from langchain.indexes import SQLRecordManager, index
from langchain_core.documents import Document
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
```
初始化一个向量存储并设置嵌入：
```python
collection_name = "test_index"
embedding = OpenAIEmbeddings()
vectorstore = ElasticsearchStore(
    es_url="http://localhost:9200", index_name="test_index", embedding=embedding
)
```
使用适当的命名空间初始化记录管理器。
**建议：** 使用一个考虑到向量存储和向量存储中的集合名称的命名空间；例如，'redis/my_docs'，'chromadb/my_docs'或者 'postgres/my_docs'。
```python
namespace = f"elasticsearch/{collection_name}"
record_manager = SQLRecordManager(
    namespace, db_url="sqlite:///record_manager_cache.sql"
)
```
在使用记录管理器之前创建一个模式。
```python
record_manager.create_schema()
```
让我们对一些测试文档进行索引：
```python
doc1 = Document(page_content="kitty", metadata={"source": "kitty.txt"})
doc2 = Document(page_content="doggy", metadata={"source": "doggy.txt"})
```
将其索引到空的向量存储中：
```python
def _clear():
    """清除内容的辅助方法。查看“full”模式部分以了解其原理。"""
    index([], record_manager, vectorstore, cleanup="full", source_id_key="source")
```
### “None”删除模式
此模式不会自动清除旧版本的内容；但仍会处理内容去重。
```python
_clear()
```
```python
index(
    [doc1, doc1, doc1, doc1, doc1],
    record_manager,
    vectorstore,
    cleanup=None,
    source_id_key="source",
)
```
```output
{'num_added': 1, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
```python
_clear()
```
```python
index([doc1, doc2], record_manager, vectorstore, cleanup=None, source_id_key="source")
```
```output
{'num_added': 2, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
第二次运行时，所有内容都将被跳过：
```python
index([doc1, doc2], record_manager, vectorstore, cleanup=None, source_id_key="source")
```
```output
{'num_added': 0, 'num_updated': 0, 'num_skipped': 2, 'num_deleted': 0}
```
### “incremental”删除模式
```python
_clear()
```
```python
index(
    [doc1, doc2],
    record_manager,
    vectorstore,
    cleanup="incremental",
    source_id_key="source",
)
```
```output
{'num_added': 2, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
再次进行索引应导致两个文档都被跳过，同时跳过嵌入操作：
```python
index(
    [doc1, doc2],
    record_manager,
    vectorstore,
    cleanup="incremental",
    source_id_key="source",
)
```
```output
{'num_added': 0, 'num_updated': 0, 'num_skipped': 2, 'num_deleted': 0}
```
如果在增量索引模式下没有提供文档，则不会发生任何变化。
```python
index([], record_manager, vectorstore, cleanup="incremental", source_id_key="source")
```
```output
{'num_added': 0, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
如果更改文档，新版本将被写入，并且所有共享相同来源的旧版本将被删除。
```python
changed_doc_2 = Document(page_content="puppy", metadata={"source": "doggy.txt"})
```
```python
index(
    [changed_doc_2],
    record_manager,
    vectorstore,
    cleanup="incremental",
    source_id_key="source",
)
```
```output
{'num_added': 1, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 1}
```
### “full”删除模式
在“full”模式下，用户应将应该被索引的“full”内容传递给索引函数。
任何未传递到索引函数中但存在于向量存储中的文档将被删除！
此行为有助于处理源文档的删除。
```python
_clear()
```
```python
all_docs = [doc1, doc2]
```
```python
index(all_docs, record_manager, vectorstore, cleanup="full", source_id_key="source")
```
假设有人删除了第一个文档：
```python
del all_docs[0]
```
```python
all_docs
```
```output
[Document(page_content='doggy', metadata={'source': 'doggy.txt'})]
```
使用全模式将清理已删除的内容。
```python
index(all_docs, record_manager, vectorstore, cleanup="full", source_id_key="source")
## 来源
metadata 属性包含一个名为 `source` 的字段。这个来源应该指向与给定文档相关的*最终*出处。
例如，如果这些文档代表某个父文档的片段，那么这两个文档的 `source` 应该是相同的，并引用父文档。
一般来说，应该始终指定 `source`。只有在您**永远**不打算使用 `incremental` 模式，并且由于某种原因无法正确指定 `source` 字段时，才使用 `None`。
```python
from langchain_text_splitters import CharacterTextSplitter
```
```python
doc1 = Document(
    page_content="kitty kitty kitty kitty kitty", metadata={"source": "kitty.txt"}
)
doc2 = Document(page_content="doggy doggy the doggy", metadata={"source": "doggy.txt"})
```
```python
new_docs = CharacterTextSplitter(
    separator="t", keep_separator=True, chunk_size=12, chunk_overlap=2
).split_documents([doc1, doc2])
new_docs
```
```output
[Document(page_content='kitty kit', metadata={'source': 'kitty.txt'}),
 Document(page_content='tty kitty ki', metadata={'source': 'kitty.txt'}),
 Document(page_content='tty kitty', metadata={'source': 'kitty.txt'}),
 Document(page_content='doggy doggy', metadata={'source': 'doggy.txt'}),
 Document(page_content='the doggy', metadata={'source': 'doggy.txt'})]
```
```python
_clear()
```
```python
index(
    new_docs,
    record_manager,
    vectorstore,
    cleanup="incremental",
    source_id_key="source",
)
```
```output
{'num_added': 5, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
```python
changed_doggy_docs = [
    Document(page_content="woof woof", metadata={"source": "doggy.txt"}),
    Document(page_content="woof woof woof", metadata={"source": "doggy.txt"}),
]
```
这将删除与 `doggy.txt` 来源相关的旧文档版本，并用新版本替换它们。
```python
index(
    changed_doggy_docs,
    record_manager,
    vectorstore,
    cleanup="incremental",
    source_id_key="source",
)
```
```output
{'num_added': 2, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 2}
```
```python
vectorstore.similarity_search("dog", k=30)
```
```output
[Document(page_content='woof woof', metadata={'source': 'doggy.txt'}),
 Document(page_content='woof woof woof', metadata={'source': 'doggy.txt'}),
 Document(page_content='tty kitty', metadata={'source': 'kitty.txt'}),
 Document(page_content='tty kitty ki', metadata={'source': 'kitty.txt'}),
 Document(page_content='kitty kit', metadata={'source': 'kitty.txt'})]
```
## 使用加载器
索引可以接受文档的可迭代对象，也可以接受任何加载器。
**注意：** 加载器**必须**正确设置来源键。
```python
from langchain_core.document_loaders import BaseLoader
class MyCustomLoader(BaseLoader):
    def lazy_load(self):
        text_splitter = CharacterTextSplitter(
            separator="t", keep_separator=True, chunk_size=12, chunk_overlap=2
        )
        docs = [
            Document(page_content="woof woof", metadata={"source": "doggy.txt"}),
            Document(page_content="woof woof woof", metadata={"source": "doggy.txt"}),
        ]
        yield from text_splitter.split_documents(docs)
    def load(self):
        return list(self.lazy_load())
```
```python
_clear()
```
```python
loader = MyCustomLoader()
```
```python
loader.load()
```
```output
[Document(page_content='woof woof', metadata={'source': 'doggy.txt'}),
 Document(page_content='woof woof woof', metadata={'source': 'doggy.txt'})]
```
```python
index(loader, record_manager, vectorstore, cleanup="full", source_id_key="source")
```
```output
{'num_added': 2, 'num_updated': 0, 'num_skipped': 0, 'num_deleted': 0}
```
```python
vectorstore.similarity_search("dog", k=30)
```
```output
[Document(page_content='woof woof', metadata={'source': 'doggy.txt'}),
 Document(page_content='woof woof woof', metadata={'source': 'doggy.txt'})]
```