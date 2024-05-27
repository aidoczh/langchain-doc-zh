# Dria

[Dria](https://dria.co/) 是一个公共 RAG 模型中心，供开发者共同贡献和利用共享的嵌入式数据湖。这篇笔记展示了如何使用 `Dria API` 进行数据检索任务。

# 安装

确保已安装 `dria` 包。您可以使用 pip 进行安装：

```python
%pip install --upgrade --quiet dria
```

# 配置 API 密钥

设置您的 Dria API 密钥以进行访问。

```python
import os
os.environ["DRIA_API_KEY"] = "DRIA_API_KEY"
```

# 初始化 Dria 检索器

创建一个 `DriaRetriever` 实例。

```python
from langchain.retrievers import DriaRetriever
api_key = os.getenv("DRIA_API_KEY")
retriever = DriaRetriever(api_key=api_key)
```

# **创建知识库**

在 [Dria 知识中心](https://dria.co/knowledge) 创建一个知识库。

```python
contract_id = retriever.create_knowledge_base(
    name="France's AI Development",
    embedding=DriaRetriever.models.jina_embeddings_v2_base_en.value,
    category="Artificial Intelligence",
    description="探索法国在人工智能领域的增长和贡献。"
)
```

# 添加数据

将数据加载到您的 Dria 知识库中。

```python
texts = [
    "要添加到 Dria 的第一段文本。",
    "另一段要存储的信息。",
    "要包含在 Dria 知识库中的更多数据。"
]
ids = retriever.add_texts(texts)
print("使用 ID 添加的数据:", ids)
```

# 检索数据

使用检索器根据查询找到相关文档。

```python
query = "查找有关 Dria 的信息。"
result = retriever.invoke(query)
for doc in result:
    print(doc)
```