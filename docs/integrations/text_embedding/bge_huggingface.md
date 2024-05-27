# Hugging Face 上的 BGE 模型

[Hugging Face 上的 BGE 模型](https://huggingface.co/BAAI/bge-large-en) 被认为是[最好的开源嵌入模型](https://huggingface.co/spaces/mteb/leaderboard)。

BGE 模型是由[北京智源人工智能研究院（BAAI）](https://en.wikipedia.org/wiki/Beijing_Academy_of_Artificial_Intelligence)创建的。`BAAI` 是一家从事人工智能研究和开发的私立非营利组织。

这个笔记展示了如何通过 `Hugging Face` 使用 `BGE Embeddings`

```python
%pip install --upgrade --quiet  sentence_transformers
```

```python
from langchain_community.embeddings import HuggingFaceBgeEmbeddings
model_name = "BAAI/bge-small-en"
model_kwargs = {"device": "cpu"}
encode_kwargs = {"normalize_embeddings": True}
hf = HuggingFaceBgeEmbeddings(
    model_name=model_name, model_kwargs=model_kwargs, encode_kwargs=encode_kwargs
)
```

```python
embedding = hf.embed_query("hi this is harrison")
len(embedding)
```

```output
384
```