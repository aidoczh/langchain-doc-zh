# 太阳能

[Solar](https://console.upstage.ai/services/embedding) 提供了嵌入服务。

这个例子演示了如何使用 LangChain 与 Solar Inference 进行文本嵌入交互。

```python
import os
os.environ["SOLAR_API_KEY"] = ""
```

```python
from langchain_community.embeddings import SolarEmbeddings
```

```python
embeddings = SolarEmbeddings()
```

```python
query_text = "这是一个测试查询。"
query_result = embeddings.embed_query(query_text)
```

```python
query_result
```

```output
[-0.009612835943698883,
 0.005192634183913469,
 -0.0007243562722578645,
 -0.02104002982378006,
 -0.004770803730934858,
 -0.024557538330554962,
 -0.03355177119374275,
 0.002088239649310708,
 0.005196372978389263,
 -0.025660645216703415,
 -0.00485575944185257,
 -0.015621133148670197,
 0.014192958362400532,
 -0.011372988112270832,
 0.02780674397945404,
 ...]
```

```python
document_text = "这是一个测试文档。"
document_result = embeddings.embed_documents([document_text])
```

```python
document_result
```

```output
[[-0.019484492018818855,
  0.0004918322083540261,
  -0.007027746178209782,
  -0.012673289515078068,
  -0.005353343673050404,
  -0.03189416974782944,
  -0.027227548882365227,
  0.0009138379828073084,
  -0.0017150233034044504,
  -0.028936535120010376,
  -0.003939046058803797,
  -0.026341330260038376,
    ...]]
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
文档和查询之间的余弦相似度：0.8685132879722154
```