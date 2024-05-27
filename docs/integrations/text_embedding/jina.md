# Jina

让我们加载 Jina 嵌入类。

```python
from langchain_community.embeddings import JinaEmbeddings
```

```python
embeddings = JinaEmbeddings(
    jina_api_key="jina_*", model_name="jina-embeddings-v2-base-en"
)
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```

```python
print(query_result)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
print(doc_result)
```