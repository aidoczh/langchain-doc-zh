# 梯度

`Gradient` 允许使用简单的网络 API 创建 `Embeddings`，并对 LLMs 进行微调和完成。

本笔记介绍如何使用 [Gradient](https://gradient.ai/) 的 Embeddings 来使用 Langchain。

## 导入

```python
from langchain_community.embeddings import GradientEmbeddings
```

## 设置环境 API 密钥

确保从 Gradient AI 获取您的 API 密钥。您将获得价值 10 美元的免费信用，用于测试和微调不同的模型。

```python
import os
from getpass import getpass
if not os.environ.get("GRADIENT_ACCESS_TOKEN", None):
    # 在 https://auth.gradient.ai/select-workspace 下获取访问令牌
    os.environ["GRADIENT_ACCESS_TOKEN"] = getpass("gradient.ai 访问令牌:")
if not os.environ.get("GRADIENT_WORKSPACE_ID", None):
    # 在 `$ gradient workspace list` 中列出的 `ID`
    # 也在登录后显示在 https://auth.gradient.ai/select-workspace
    os.environ["GRADIENT_WORKSPACE_ID"] = getpass("gradient.ai 工作空间 ID:")
```

可选：验证您的环境变量 `GRADIENT_ACCESS_TOKEN` 和 `GRADIENT_WORKSPACE_ID` 以获取当前部署的模型。使用 `gradientai` Python 包。

```python
%pip install --upgrade --quiet  gradientai
```

## 创建 Gradient 实例

```python
documents = [
    "Pizza is a dish.",
    "Paris is the capital of France",
    "numpy is a lib for linear algebra",
]
query = "Where is Paris?"
```

```python
embeddings = GradientEmbeddings(model="bge-large")
documents_embedded = embeddings.embed_documents(documents)
query_result = embeddings.embed_query(query)
```

```python
# (演示) 计算相似度
import numpy as np
scores = np.array(documents_embedded) @ np.array(query_result).T
dict(zip(documents, scores))
```