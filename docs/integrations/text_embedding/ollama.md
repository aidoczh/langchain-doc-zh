# Ollama

让我们加载 Ollama Embeddings 类。

```python
from langchain_community.embeddings import OllamaEmbeddings
```

```python
embeddings = OllamaEmbeddings()
```

```python
text = "这是一个测试文档。"
```

要生成嵌入向量，你可以查询单个文本，也可以查询文本列表。

```python
query_result = embeddings.embed_query(text)
query_result[:5]
```

```output
[-0.09996652603149414,
 0.015568195842206478,
 0.17670190334320068,
 0.16521021723747253,
 0.21193109452724457]
```

```python
doc_result = embeddings.embed_documents([text])
doc_result[0][:5]
```

```output
[-0.04242777079343796,
 0.016536075621843338,
 0.10052520781755447,
 0.18272875249385834,
 0.2079043835401535]
```

让我们使用较小的模型（例如 llama:7b）加载 Ollama Embeddings 类。注意：请参阅其他支持的模型 [https://ollama.ai/library](https://ollama.ai/library)

```python
embeddings = OllamaEmbeddings(model="llama2:7b")
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```

```python
query_result[:5]
```

```output
[-0.09996627271175385,
 0.015567859634757042,
 0.17670205235481262,
 0.16521376371383667,
 0.21193283796310425]
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
doc_result[0][:5]
```

```output
[-0.042427532374858856,
 0.01653730869293213,
 0.10052604228258133,
 0.18272635340690613,
 0.20790338516235352]
```