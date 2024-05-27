---

sidebar_label: Together AI

---

# TogetherEmbeddings

这份笔记覆盖了如何开始使用托管在 Together AI API 中的开源嵌入模型。

## 安装

```python
# 安装包
%pip install --upgrade --quiet  langchain-together
```

## 环境设置

确保设置以下环境变量：

- `TOGETHER_API_KEY`

## 使用

首先，从[这个列表](https://docs.together.ai/docs/embedding-models)中选择一个支持的模型。在下面的示例中，我们将使用 `togethercomputer/m2-bert-80M-8k-retrieval` 模型。

```python
from langchain_together.embeddings import TogetherEmbeddings
embeddings = TogetherEmbeddings(model="togethercomputer/m2-bert-80M-8k-retrieval")
```

```python
embeddings.embed_query("我的查询内容")
```

```python
embeddings.embed_documents(
    ["这是一个文档的内容", "这是另一个文档"]
)
```

```python
# 异步嵌入查询
await embeddings.aembed_query("我的查询内容")
```

```python
# 异步嵌入文档
await embeddings.aembed_documents(
    ["这是一个文档的内容", "这是另一个文档"]
)
```