---

sidebar_label: Nomic

---

# NomicEmbeddings

本笔记介绍了如何开始使用 Nomic 嵌入模型。

## 安装

```python
# 安装包
!pip install -U langchain-nomic
```

## 环境设置

确保设置以下环境变量：

- `NOMIC_API_KEY`

## 使用

```python
from langchain_nomic.embeddings import NomicEmbeddings
embeddings = NomicEmbeddings(model="nomic-embed-text-v1.5")
```

```python
embeddings.embed_query("My query to look up")
```

```python
embeddings.embed_documents(
    ["This is a content of the document", "This is another document"]
)
```

```python
# 异步嵌入查询
await embeddings.aembed_query("My query to look up")
```

```python
# 异步嵌入文档
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```

### 自定义维度

Nomic 的 `nomic-embed-text-v1.5` 模型是通过[Matryoshka learning 进行训练的](https://blog.nomic.ai/posts/nomic-embed-matryoshka)，以实现单个模型的可变长度嵌入。这意味着您可以在推断时指定嵌入的维度。该模型支持从 64 到 768 的维度。

```python
embeddings = NomicEmbeddings(model="nomic-embed-text-v1.5", dimensionality=256)
embeddings.embed_query("My query to look up")
```
