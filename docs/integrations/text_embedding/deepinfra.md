# DeepInfra

[DeepInfra](https://deepinfra.com/?utm_source=langchain) 是一种无服务器推理服务，提供对[各种LLM模型](https://deepinfra.com/models?utm_source=langchain)和[嵌入模型](https://deepinfra.com/models?type=embeddings&utm_source=langchain)的访问。本笔记介绍了如何使用 LangChain 结合 DeepInfra 进行文本嵌入。

```python
# 注册账户：https://deepinfra.com/login?utm_source=langchain
from getpass import getpass
DEEPINFRA_API_TOKEN = getpass()
```

```output
 ········
```

```python
import os
os.environ["DEEPINFRA_API_TOKEN"] = DEEPINFRA_API_TOKEN
```

```python
from langchain_community.embeddings import DeepInfraEmbeddings
```

```python
embeddings = DeepInfraEmbeddings(
    model_id="sentence-transformers/clip-ViT-B-32",
    query_instruction="",
    embed_instruction="",
)
```

```python
docs = ["Dog is not a cat", "Beta is the second letter of Greek alphabet"]
document_result = embeddings.embed_documents(docs)
```

```python
query = "What is the first letter of Greek alphabet"
query_result = embeddings.embed_query(query)
```

```python
import numpy as np
query_numpy = np.array(query_result)
for doc_res, doc in zip(document_result, docs):
    document_numpy = np.array(doc_res)
    similarity = np.dot(query_numpy, document_numpy) / (
        np.linalg.norm(query_numpy) * np.linalg.norm(document_numpy)
    )
    print(f'"{doc}" 与查询的余弦相似度：{similarity}')
```

```output
"Dog is not a cat" 与查询的余弦相似度：0.7489097144129355
"Beta is the second letter of Greek alphabet" 与查询的余弦相似度：0.9519380640702013
```