# 阿莱夫·阿尔法

阿莱夫·阿尔法的语义嵌入有两种可能的使用方式。如果您有结构不同的文本（例如文档和查询），您会希望使用不对称嵌入。相反，对于结构相似的文本，建议采用对称嵌入。

## 不对称嵌入

```python
from langchain_community.embeddings import AlephAlphaAsymmetricSemanticEmbedding
```

```python
document = "This is a content of the document"
query = "What is the content of the document?"
```

```python
embeddings = AlephAlphaAsymmetricSemanticEmbedding(normalize=True, compress_to_size=128)
```

```python
doc_result = embeddings.embed_documents([document])
```

```python
query_result = embeddings.embed_query(query)
```

## 对称嵌入

```python
from langchain_community.embeddings import AlephAlphaSymmetricSemanticEmbedding
```

```python
text = "This is a test text"
```

```python
embeddings = AlephAlphaSymmetricSemanticEmbedding(normalize=True, compress_to_size=128)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
query_result = embeddings.embed_query(text)
```