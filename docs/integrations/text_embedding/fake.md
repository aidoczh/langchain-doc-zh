# 虚假嵌入

LangChain 还提供了一个虚假嵌入类。您可以使用这个类来测试您的流水线。

```python
from langchain_community.embeddings import FakeEmbeddings
```

```python
embeddings = FakeEmbeddings(size=1352)
```

```python
query_result = embeddings.embed_query("foo")
```

```python
doc_results = embeddings.embed_documents(["foo"])
```