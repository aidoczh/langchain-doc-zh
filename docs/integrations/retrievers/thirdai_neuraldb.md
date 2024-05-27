# **NeuralDB**

NeuralDB 是由 ThirdAI 开发的一款 CPU 友好且可微调的检索引擎。

### **初始化**

有两种初始化方法：

- 从头开始：基本模型

- 从检查点开始：加载先前保存的模型

对于以下所有初始化方法，如果设置了 `THIRDAI_KEY` 环境变量，则可以省略 `thirdai_key` 参数。

ThirdAI API 密钥可在 https://www.thirdai.com/try-bolt/ 获取。

```python
from langchain.retrievers import NeuralDBRetriever
# 从头开始
retriever = NeuralDBRetriever.from_scratch(thirdai_key="your-thirdai-key")
# 从检查点开始
retriever = NeuralDBRetriever.from_checkpoint(
    # NeuralDB 检查点的路径。例如，如果您在一个脚本中调用
    # retriever.save("/path/to/checkpoint.ndb")，那么您可以在另一个脚本中调用
    # NeuralDBRetriever.from_checkpoint("/path/to/checkpoint.ndb") 来加载保存的模型。
    checkpoint="/path/to/checkpoint.ndb",
    thirdai_key="your-thirdai-key",
)
```

### **插入文档源**

```python
retriever.insert(
    # 如果您有 PDF、DOCX 或 CSV 文件，可以直接传递文档的路径
    sources=["/path/to/doc.pdf", "/path/to/doc.docx", "/path/to/doc.csv"],
    # 当为 True 时，意味着 NeuralDB 中的基础模型将对插入的文件进行无监督预训练。默认为 True。
    train=True,
    # 在性能略微下降的情况下更快地插入。默认为 True。
    fast_mode=True,
)
from thirdai import neural_db as ndb
retriever.insert(
    # 如果您有其他格式的文件，或者更喜欢配置文件的解析方式，
    # 则可以像这样传递 NeuralDB 文档对象。
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

### **检索文档**

要查询检索器，可以使用标准的 LangChain 检索器方法 `get_relevant_documents`，该方法返回一个 LangChain 文档对象列表。每个文档对象代表来自索引文件的文本块。例如，它可能包含来自索引的 PDF 文件中的一个段落。除了文本外，文档的元数据字段还包含诸如文档 ID、文档来源（来自哪个文件）和文档分数等信息。

```python
# 这将返回一个 LangChain 文档对象列表
documents = retriever.invoke("query", top_k=10)
```

### **微调**

NeuralDBRetriever 可以根据用户行为和领域特定知识进行微调。它可以通过两种方式进行微调：

1. 关联：检索器将源短语与目标短语关联起来。当检索器看到源短语时，它还将考虑与目标短语相关的结果。

2. 点赞：检索器提高特定查询的文档分数。当您希望根据用户行为微调检索器时，这很有用。例如，如果用户搜索“汽车是如何制造的”并喜欢返回的文档 ID 52，则我们可以为查询“汽车是如何制造的”点赞文档 ID 52。

```python
retriever.associate(source="source phrase", target="target phrase")
retriever.associate_batch(
    [
        ("source phrase 1", "target phrase 1"),
        ("source phrase 2", "target phrase 2"),
    ]
)
retriever.upvote(query="how is a car manufactured", document_id=52)
retriever.upvote_batch(
    [
        ("query 1", 52),
        ("query 2", 20),
    ]
)
```