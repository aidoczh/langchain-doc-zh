# FireworksEmbeddings

本文介绍如何使用 Fireworks Embeddings，它包含在 langchain_fireworks 软件包中，用于在 langchain 中嵌入文本。在本示例中，我们使用默认的 nomic-ai v1.5 模型。

```python
%pip install -qU langchain-fireworks
```

## 设置

```python
from langchain_fireworks import FireworksEmbeddings
```

```python
import getpass
import os
if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")
```

# 使用嵌入模型

使用 `FireworksEmbeddings`，您可以直接使用默认模型'nomic-ai/nomic-embed-text-v1.5'，或者如果可用，设置为其他模型。

```python
embedding = FireworksEmbeddings(model="nomic-ai/nomic-embed-text-v1.5")
```

```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
print(res_query[:5])
print(res_document[1][:5])
```

```output
[0.01367950439453125, 0.0103607177734375, -0.157958984375, -0.003070831298828125, 0.05926513671875]
[0.0369873046875, 0.00545501708984375, -0.179931640625, -0.018707275390625, 0.0552978515625]
```