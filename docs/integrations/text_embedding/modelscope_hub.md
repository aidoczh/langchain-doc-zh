# ModelScope

[ModelScope](https://www.modelscope.cn/home) 是一个大型的模型和数据集仓库。

让我们加载 ModelScope Embedding 类。

```python
from langchain_community.embeddings import ModelScopeEmbeddings
```

```python
model_id = "damo/nlp_corom_sentence-embedding_english-base"
```

```python
embeddings = ModelScopeEmbeddings(model_id=model_id)
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_results = embeddings.embed_documents(["foo"])
```