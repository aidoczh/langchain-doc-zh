# Hugging Face 上的 Instruct Embeddings

[Hugging Face 句子转换器](https://huggingface.co/sentence-transformers) 是一个用于最先进的句子、文本和图像嵌入的 Python 框架。

其中的 instruct 嵌入模型被用于 `HuggingFaceInstructEmbeddings` 类中。

```python
from langchain_community.embeddings import HuggingFaceInstructEmbeddings
```

```python
embeddings = HuggingFaceInstructEmbeddings(
    query_instruction="表示用于检索的查询："
)
```

```output
load INSTRUCTOR_Transformer
max_seq_length  512
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```