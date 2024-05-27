# 如何使用基于时间加权向量存储的检索器

这种检索器结合了语义相似度和时间衰减。

评分算法如下：

```
语义相似度 + (1.0 - 衰减率) ^ 经过的小时数
```

值得注意的是，`经过的小时数` 指的是自从检索器中的对象**上次被访问**以来经过的小时数，而不是自创建以来经过的小时数。这意味着频繁访问的对象保持“新鲜”。

```python
from datetime import datetime, timedelta
import faiss
from langchain.retrievers import TimeWeightedVectorStoreRetriever
from langchain_community.docstore import InMemoryDocstore
from langchain_community.vectorstores import FAISS
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
```

## 低衰减率

低的 `衰减率`（在这里，为了极端，我们将其设定接近于0）意味着记忆将被“记住”更长时间。 衰减率为0意味着记忆永远不会被遗忘，使得这个检索器等同于向量查找。

```python
# 定义您的嵌入模型
embeddings_model = OpenAIEmbeddings()
# 将向量存储初始化为空
embedding_size = 1536
index = faiss.IndexFlatL2(embedding_size)
vectorstore = FAISS(embeddings_model, index, InMemoryDocstore({}), {})
retriever = TimeWeightedVectorStoreRetriever(
    vectorstore=vectorstore, decay_rate=0.0000000000000000000000001, k=1
)
```

```python
yesterday = datetime.now() - timedelta(days=1)
retriever.add_documents(
    [Document(page_content="hello world", metadata={"last_accessed_at": yesterday})]
)
retriever.add_documents([Document(page_content="hello foo")])
```

```output
['c3dcf671-3c0a-4273-9334-c4a913076bfa']
```

```python
# "Hello World" 被首先返回，因为它最显著，而衰减率接近于0，意味着它仍然足够新鲜
retriever.get_relevant_documents("hello world")
```

```output
[Document(page_content='hello world', metadata={'last_accessed_at': datetime.datetime(2023, 12, 27, 15, 30, 18, 457125), 'created_at': datetime.datetime(2023, 12, 27, 15, 30, 8, 442662), 'buffer_idx': 0})]
```

## 高衰减率

当 `衰减率` 较高（例如，多个9），`最近分数` 快速降至0！ 如果将其设置为1，所有对象的 `最近性` 都为0，再次使其等同于向量查找。

```python
# 定义您的嵌入模型
embeddings_model = OpenAIEmbeddings()
# 将向量存储初始化为空
embedding_size = 1536
index = faiss.IndexFlatL2(embedding_size)
vectorstore = FAISS(embeddings_model, index, InMemoryDocstore({}), {})
retriever = TimeWeightedVectorStoreRetriever(
    vectorstore=vectorstore, decay_rate=0.999, k=1
)
```

```python
yesterday = datetime.now() - timedelta(days=1)
retriever.add_documents(
    [Document(page_content="hello world", metadata={"last_accessed_at": yesterday})]
)
retriever.add_documents([Document(page_content="hello foo")])
```

```output
['eb1c4c86-01a8-40e3-8393-9a927295a950']
```

```python
# "Hello Foo" 被首先返回，因为 "hello world" 已经大部分被遗忘
retriever.get_relevant_documents("hello world")
```

```output
[Document(page_content='hello foo', metadata={'last_accessed_at': datetime.datetime(2023, 12, 27, 15, 30, 50, 57185), 'created_at': datetime.datetime(2023, 12, 27, 15, 30, 44, 720490), 'buffer_idx': 1})]
```

## 虚拟时间

使用 LangChain 中的一些工具，您可以模拟时间组件。

```python
import datetime
from langchain_core.utils import mock_now
```

```python
# 注意最后访问时间是该日期时间
with mock_now(datetime.datetime(2024, 2, 3, 10, 11)):
    print(retriever.get_relevant_documents("hello world"))
```

```output
[Document(page_content='hello world', metadata={'last_accessed_at': MockDateTime(2024, 2, 3, 10, 11), 'created_at': datetime.datetime(2023, 12, 27, 15, 30, 44, 532941), 'buffer_idx': 0})]
```