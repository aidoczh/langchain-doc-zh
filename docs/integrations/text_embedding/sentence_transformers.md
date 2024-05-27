# Hugging Face 上的句子转换器

[Hugging Face 句子转换器](https://huggingface.co/sentence-transformers) 是一个用于最先进的句子、文本和图像嵌入的 Python 框架。

其中一个嵌入模型被用在 `HuggingFaceEmbeddings` 类中。

我们还为那些更熟悉直接使用该软件包的用户添加了 `SentenceTransformerEmbeddings` 的别名。

`sentence_transformers` 软件包的模型源自 [Sentence-BERT](https://arxiv.org/abs/1908.10084)。

```python
%pip install --upgrade --quiet  sentence_transformers > /dev/null
```

```output
[notice] A new release of pip is available: 23.0.1 -> 23.1.1
[notice] To update, run: pip install --upgrade pip
```

```python
from langchain_huggingface import HuggingFaceEmbeddings
```

```python
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
# 等同于 SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_result = embeddings.embed_documents([text, "This is not a test document."])
```