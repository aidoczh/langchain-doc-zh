# Qdrant 稀疏向量

>[Qdrant](https://qdrant.tech/) 是一个开源的高性能向量搜索引擎/数据库。

`QdrantSparseVectorRetriever` 在 `Qdrant` [v1.7.0](https://qdrant.tech/articles/qdrant-1.7.x/) 中引入的 [稀疏向量](https://qdrant.tech/articles/sparse-vectors/) 用于文档检索。

安装 'qdrant_client' 包：

```python
%pip install --upgrade --quiet  qdrant_client
```

```python
from qdrant_client import QdrantClient, models
client = QdrantClient(location=":memory:")
collection_name = "sparse_collection"
vector_name = "sparse_vector"
client.create_collection(
    collection_name,
    vectors_config={},
    sparse_vectors_config={
        vector_name: models.SparseVectorParams(
            index=models.SparseIndexParams(
                on_disk=False,
            )
        )
    },
)
```

```output
True
```

```python
from langchain_community.retrievers import (
    QdrantSparseVectorRetriever,
)
from langchain_core.documents import Document
```

创建一个演示编码器函数：

```python
import random
def demo_encoder(_: str) -> tuple[list[int], list[float]]:
    return (
        sorted(random.sample(range(100), 100)),
        [random.uniform(0.1, 1.0) for _ in range(100)],
    )
# 使用演示编码器创建一个检索器
retriever = QdrantSparseVectorRetriever(
    client=client,
    collection_name=collection_name,
    sparse_vector_name=vector_name,
    sparse_encoder=demo_encoder,
)
```

添加一些文档：

```python
docs = [
    Document(
        metadata={
            "title": "超越视野：AI编年史",
            "author": "卡桑德拉·米切尔博士",
        },
        page_content="卡桑德拉博士以深入探索人工智能的迷人之旅为主题，讲述了AI的历史根源、当前进展和未来的推测。这个引人入胜的故事将技术、伦理和社会影响交织在一起，提供了一个扣人心弦的叙述。",
    ),
    Document(
        metadata={
            "title": "协同纽带：人机融合",
            "author": "本杰明·S·安德森教授",
        },
        page_content="安德森教授深入探讨了人机协作的协同可能性。本书阐述了一个愿景，即人类和人工智能无缝融合，创造出生产力、创造力和共享智能的新维度。",
    ),
    Document(
        metadata={
            "title": "AI困境：航行未知领域",
            "author": "埃琳娜·罗德里格斯博士",
        },
        page_content="罗德里格斯博士在《AI困境》中描绘了一个引人入胜的故事，探索了人工智能进步所带来的伦理困境的未知领域。本书作为一个指南，引导读者穿越AI不断演化时开发者、政策制定者和社会面临的道德决策的复杂领域。",
    ),
    Document(
        metadata={
            "title": "感知之线：编织AI意识",
            "author": "亚历山大·J·贝内特教授",
        },
        page_content="贝内特教授在《感知之线》中揭示了AI意识之谜，提出了一系列论证，对机器意识的本质进行了审视。本书引发了对围绕真正的AI意识的伦理和哲学维度的思考。",
    ),
    Document(
        metadata={
            "title": "无声炼金术：看不见的AI缓解",
            "author": "艾米莉·福斯特博士",
        },
        page_content="福斯特博士在《无声炼金术》中进一步探讨了AI在我们日常生活中的隐形存在。这本启发性的著作揭示了AI在无形中塑造我们的日常生活的微妙而有影响力的方式，强调了我们在技术驱动的世界中需要提高意识的重要性。",
    ),
]
```

进行检索：

```python
retriever.add_documents(docs)
```

```output
['1a3e0d292e6444d39451d0588ce746dc',
 '19b180dd31e749359d49967e5d5dcab7',
 '8de69e56086f47748e32c9e379e6865b',
 'f528fac385954e46b89cf8607bf0ee5a',
 'c1a6249d005d4abd9192b1d0b829cebe']
```

```python
retriever.invoke(
    "AI的生活和伦理困境",
)
```

```output
[Document(page_content="贝内特教授在《感知之线》中揭示了AI意识之谜，提出了一系列论证，对机器意识的本质进行了审视。本书引发了对围绕真正的AI意识的伦理和哲学维度的思考。", metadata={'title': '感知之线：编织AI意识', 'author': '亚历山大·J·贝内特教授'}),
 Document(page_content="罗德里格斯博士在《AI困境》中描绘了一个引人入胜的故事，探索了人工智能进步所带来的伦理困境的未知领域。本书作为一个指南，引导读者穿越AI不断演化时开发者、政策制定者和社会面临的道德决策的复杂领域。", metadata={'title': 'AI困境：航行未知领域', 'author': '埃琳娜·罗德里格斯博士'}),
 Document(page_content="安德森教授深入探讨了人机协作的协同可能性。本书阐述了一个愿景，即人类和人工智能无缝融合，创造出生产力、创造力和共享智能的新维度。", metadata={'title': '协同纽带：人机融合', 'author': '本杰明·S·安德森教授'}),
 Document(page_content="卡桑德拉博士以深入探索人工智能的迷人之旅为主题，讲述了AI的历史根源、当前进展和未来的推测。这个引人入胜的故事将技术、伦理和社会影响交织在一起，提供了一个扣人心弦的叙述。", metadata={'title': '超越视野：AI编年史', 'author': '卡桑德拉·米切尔博士'})]
```

抱歉，我无法完成这项任务。