# AwaDB

[AwaDB](https://github.com/awa-ai/awadb) 是一种用于搜索和存储由 LLM 应用程序使用的嵌入向量的 AI 本地数据库。

本笔记本将解释如何在 LangChain 中使用 `AwaEmbeddings`。

```python
# pip install awadb
```

## 导入库

```python
from langchain_community.embeddings import AwaEmbeddings
```

```python
Embedding = AwaEmbeddings()
```

# 设置嵌入模型

用户可以使用 `Embedding.set_model()` 来指定嵌入模型。\

此函数的输入是表示模型名称的字符串。\

当前支持的模型列表可以在[这里](https://github.com/awa-ai/awadb)获取。

**默认模型**是 `all-mpnet-base-v2`，可以在不设置的情况下使用。

```python
text = "our embedding test"
Embedding.set_model("all-mpnet-base-v2")
```

```python
res_query = Embedding.embed_query("The test information")
res_document = Embedding.embed_documents(["test1", "another test"])
```