# MiniMax

[MiniMax](https://api.minimax.chat/document/guides/embeddings?id=6464722084cdc277dfaa966a) 提供嵌入式服务。

这个示例演示了如何使用 LangChain 与 MiniMax 推理进行文本嵌入交互。

```python
import os
os.environ["MINIMAX_GROUP_ID"] = "MINIMAX_GROUP_ID"
os.environ["MINIMAX_API_KEY"] = "MINIMAX_API_KEY"
```

```python
from langchain_community.embeddings import MiniMaxEmbeddings
```

```python
embeddings = MiniMaxEmbeddings()
```

```python
query_text = "这是一个测试查询。"
query_result = embeddings.embed_query(query_text)
```

```python
document_text = "这是一个测试文档。"
document_result = embeddings.embed_documents([document_text])
```

```python
import numpy as np
query_numpy = np.array(query_result)
document_numpy = np.array(document_result[0])
similarity = np.dot(query_numpy, document_numpy) / (
    np.linalg.norm(query_numpy) * np.linalg.norm(document_numpy)
)
print(f"文档和查询之间的余弦相似度：{similarity}")
```

```output
文档和查询之间的余弦相似度：0.1573236279277012
```