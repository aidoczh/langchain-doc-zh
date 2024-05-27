# 基石

>[亚马逊基石](https://aws.amazon.com/bedrock/)是一项完全托管的服务，提供了来自领先人工智能公司如`AI21 Labs`、`Anthropic`、`Cohere`、`Meta`、`Stability AI`和`Amazon`的高性能基础模型（FMs），通过单一 API，以及您构建生成式人工智能应用所需的广泛功能，包括安全性、隐私性和负责任的人工智能。使用`亚马逊基石`，您可以轻松尝试和评估适合您用例的顶级 FMs，使用微调和`检索增强生成`（`RAG`）等技术私密定制它们，并构建执行任务的代理，这些代理使用您的企业系统和数据源。由于`亚马逊基石`是无服务器的，您无需管理任何基础设施，可以安全地集成和部署生成式人工智能功能到您已熟悉的 AWS 服务中。

```python
%pip install --upgrade --quiet  boto3
```

```python
from langchain_community.embeddings import BedrockEmbeddings
embeddings = BedrockEmbeddings(
    credentials_profile_name="bedrock-admin", region_name="us-east-1"
)
```

```python
embeddings.embed_query("这是文档的内容")
```

```python
embeddings.embed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
```

```python
# 异步嵌入查询
await embeddings.aembed_query("这是文档的内容")
```

```python
# 异步嵌入文档
await embeddings.aembed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
```