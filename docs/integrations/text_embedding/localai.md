# 本地AI

让我们加载 LocalAI 嵌入类。为了使用 LocalAI 嵌入类，你需要在某个地方托管 LocalAI 服务并配置嵌入模型。请参阅文档 https://localai.io/basics/getting_started/index.html 和 https://localai.io/features/embeddings/index.html。

```python
from langchain_community.embeddings import LocalAIEmbeddings
```

```python
embeddings = LocalAIEmbeddings(
    openai_api_base="http://localhost:8080", model="embedding-model-name"
)
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_result = embeddings.embed_documents([text])
```

让我们使用第一代模型加载 LocalAI 嵌入类（例如 text-search-ada-doc-001/text-search-ada-query-001）。注意：这些不是推荐的模型 - 请参阅[这里](https://platform.openai.com/docs/guides/embeddings/what-are-embeddings)

```python
from langchain_community.embeddings import LocalAIEmbeddings
```

```python
embeddings = LocalAIEmbeddings(
    openai_api_base="http://localhost:8080", model="embedding-model-name"
)
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
import os
# 如果你在显式代理后面，你可以使用 OPENAI_PROXY 环境变量来传递
os.environ["OPENAI_PROXY"] = "http://proxy.yourcompany.com:8080"
```