# MistralAI

本文介绍如何使用 MistralAIEmbeddings，在 langchain_mistralai 包中提供的功能，将文本嵌入到 langchain 中。

```python
# pip install -U langchain-mistralai
```

## 导入库

```python
from langchain_mistralai import MistralAIEmbeddings
```

```python
embedding = MistralAIEmbeddings(api_key="your-api-key")
```

# 使用嵌入模型

使用 `MistralAIEmbeddings`，您可以直接使用默认模型 'mistral-embed'，或者如果有其他可用模型，也可以设置为其他模型。

```python
embedding.model = "mistral-embed"  # 或者您偏好的其他模型（如果可用）
```

```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
```