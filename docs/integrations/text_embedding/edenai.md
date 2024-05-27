# EDEN AI

Eden AI正在通过整合最优秀的AI提供商，赋予用户解锁无限可能性的能力，发挥人工智能的真正潜力，从而彻底改变人工智能领域。通过一体化全面且无障碍的平台，用户可以快速将AI功能部署到生产环境，通过单一API轻松访问完整的AI功能范围。 (网站: [https://edenai.co/](https://edenai.co/))

以下示例介绍了如何使用LangChain与Eden AI嵌入模型进行交互

-----------------------------------------------------------------------------------

要访问EDENAI的API，需要一个API密钥，您可以通过创建帐户 [https://app.edenai.run/user/register](https://app.edenai.run/user/register) 并前往 [https://app.edenai.run/admin/account/settings](https://app.edenai.run/admin/account/settings) 获取。

一旦获得密钥，我们希望将其设置为环境变量，方法是运行:

```shell
export EDENAI_API_KEY="..."
```

如果您不想设置环境变量，可以直接通过edenai_api_key命名参数传递密钥，在初始化EdenAI嵌入类时:

```python
from langchain_community.embeddings.edenai import EdenAiEmbeddings
```

```python
embeddings = EdenAiEmbeddings(edenai_api_key="...", provider="...")
```

## 调用模型

EdenAI API汇集了各种提供商。

要访问特定模型，只需在调用时使用"provider"。

```python
embeddings = EdenAiEmbeddings(provider="openai")
```

```python
docs = ["现在正在下雨", "猫咪很可爱"]
document_result = embeddings.embed_documents(docs)
```

```python
query = "我的雨伞坏了"
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
    print(f'"{doc}" 与查询之间的余弦相似度: {similarity}')
```

```output
"现在正在下雨" 与查询之间的余弦相似度: 0.849261496107252
"猫咪很可爱" 与查询之间的余弦相似度: 0.7525900655705218
```