---

sidebar_label: Upstage

---

# UpstageEmbeddings

本文介绍了如何使用 Upstage 嵌入模型。

## 安装

安装 `langchain-upstage` 包。

```bash
pip install -U langchain-upstage
```

## 环境设置

确保设置以下环境变量：

- `UPSTAGE_API_KEY`：从 [Upstage 控制台](https://console.upstage.ai/) 获取的 Upstage API 密钥。

```python
import os
os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```

## 使用方法

初始化 `UpstageEmbeddings` 类。

```python
from langchain_upstage import UpstageEmbeddings
embeddings = UpstageEmbeddings()
```

使用 `embed_documents` 方法嵌入文本或文档列表。

```python
doc_result = embeddings.embed_documents(
    ["Sam is a teacher.", "This is another document"]
)
print(doc_result)
```

使用 `embed_query` 方法嵌入查询字符串。

```python
query_result = embeddings.embed_query("What does Sam do?")
print(query_result)
```

使用 `aembed_documents` 和 `aembed_query` 进行异步操作。

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

## 与向量存储一起使用

您可以将 `UpstageEmbeddings` 与向量存储组件一起使用。以下是一个简单示例。

```python
from langchain_community.vectorstores import DocArrayInMemorySearch
vectorstore = DocArrayInMemorySearch.from_texts(
    ["harrison worked at kensho", "bears like to eat honey"],
    embedding=UpstageEmbeddings(),
)
retriever = vectorstore.as_retriever()
docs = retriever.invoke("Where did Harrison work?")
print(docs)
```
