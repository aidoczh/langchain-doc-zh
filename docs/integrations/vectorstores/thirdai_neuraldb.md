# ThirdAI NeuralDB

[NeuralDB](https://www.thirdai.com/neuraldb-enterprise/) 是由 [ThirdAI](https://www.thirdai.com/) 开发的 CPU 友好且可微调的向量存储。

## 初始化

有两种初始化方法：

- 从头开始：基本模型

- 从检查点开始：加载先前保存的模型

对于以下所有初始化方法，如果设置了 `THIRDAI_KEY` 环境变量，则可以省略 `thirdai_key` 参数。

ThirdAI API 密钥可在 https://www.thirdai.com/try-bolt/ 获取。

```python
from langchain.vectorstores import NeuralDBVectorStore
# 从头开始
vectorstore = NeuralDBVectorStore.from_scratch(thirdai_key="your-thirdai-key")
# 从检查点开始
vectorstore = NeuralDBVectorStore.from_checkpoint(
    # NeuralDB 检查点的路径。例如，如果您在一个脚本中调用
    # vectorstore.save("/path/to/checkpoint.ndb")，那么您可以在另一个脚本中调用
    # NeuralDBVectorStore.from_checkpoint("/path/to/checkpoint.ndb") 来加载保存的模型。
    checkpoint="/path/to/checkpoint.ndb",
    thirdai_key="your-thirdai-key",
)
```

## 插入文档来源

```python
vectorstore.insert(
    # 如果您有 PDF、DOCX 或 CSV 文件，可以直接传递文档的路径
    sources=["/path/to/doc.pdf", "/path/to/doc.docx", "/path/to/doc.csv"],
    # 当为 True 时，意味着 NeuralDB 中的基础模型将对插入的文件进行无监督预训练。默认为 True。
    train=True,
    # 以稍微降低性能为代价实现更快的插入。默认为 True。
    fast_mode=True,
)
from thirdai import neural_db as ndb
vectorstore.insert(
    # 如果您有其他格式的文件，或者更喜欢配置文件的解析方式，
    # 那么可以像这样传递 NeuralDB 文档对象。
    sources=[
        ndb.PDF(
            "/path/to/doc.pdf",
            version="v2",
            chunk_size=100,
            metadata={"published": 2022},
        ),
        ndb.Unstructured("/path/to/deck.pptx"),
    ]
)
```

## 相似度搜索

要查询向量存储，可以使用标准的 LangChain 向量存储方法 `similarity_search`，它返回一个 LangChain 文档对象的列表。每个文档对象代表来自索引文件的文本块。例如，它可能包含来自索引的 PDF 文件中的一个段落。除了文本外，文档的元数据字段还包含诸如文档的 ID、该文档的来源（它来自哪个文件）以及文档的分数等信息。

```python
# 这将返回一个 LangChain 文档对象的列表
documents = vectorstore.similarity_search("query", k=10)
```

## 微调

NeuralDBVectorStore 可以根据用户行为和领域特定知识进行微调。它可以通过两种方式进行微调：

1. 关联：向量存储将源短语与目标短语关联起来。当向量存储看到源短语时，它还将考虑与目标短语相关的结果。

2. 点赞：向量存储提高特定查询的文档分数。当您希望将向量存储微调到用户行为时，这将非常有用。例如，如果用户搜索“汽车是如何制造的”并喜欢返回的 ID 为 52 的文档，则我们可以为查询“汽车是如何制造的”点赞 ID 为 52 的文档。

```python
vectorstore.associate(source="source phrase", target="target phrase")
vectorstore.associate_batch(
    [
        ("source phrase 1", "target phrase 1"),
        ("source phrase 2", "target phrase 2"),
    ]
)
vectorstore.upvote(query="how is a car manufactured", document_id=52)
vectorstore.upvote_batch(
    [
        ("query 1", 52),
        ("query 2", 20),
    ]
)
```