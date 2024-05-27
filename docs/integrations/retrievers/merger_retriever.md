# 魔戒（Merger Retriever）

>`检索者之主（LOTR）`，也被称为`MergerRetriever`，接受一个检索者列表作为输入，并将它们的`get_relevant_documents()`方法的结果合并成一个列表。合并后的结果将是一个与查询相关且已由不同检索者排名的文档列表。

`MergerRetriever`类可用于多种方式来提高文档检索的准确性。首先，它可以合并多个检索者的结果，有助于减少结果中的偏见风险。其次，它可以对不同检索者的结果进行排名，有助于确保最相关的文档被优先返回。

```python
import os
import chromadb
from langchain.retrievers import (
    ContextualCompressionRetriever,
    DocumentCompressorPipeline,
    MergerRetriever,
)
from langchain_chroma import Chroma
from langchain_community.document_transformers import (
    EmbeddingsClusteringFilter,
    EmbeddingsRedundantFilter,
)
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_openai import OpenAIEmbeddings
# 获取3种不同的嵌入向量。
all_mini = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
multi_qa_mini = HuggingFaceEmbeddings(model_name="multi-qa-MiniLM-L6-dot-v1")
filter_embeddings = OpenAIEmbeddings()
ABS_PATH = os.path.dirname(os.path.abspath(__file__))
DB_DIR = os.path.join(ABS_PATH, "db")
# 实例化2个不同的chromadb索引，每个索引都使用不同的嵌入向量。
client_settings = chromadb.config.Settings(
    is_persistent=True,
    persist_directory=DB_DIR,
    anonymized_telemetry=False,
)
db_all = Chroma(
    collection_name="project_store_all",
    persist_directory=DB_DIR,
    client_settings=client_settings,
    embedding_function=all_mini,
)
db_multi_qa = Chroma(
    collection_name="project_store_multi",
    persist_directory=DB_DIR,
    client_settings=client_settings,
    embedding_function=multi_qa_mini,
)
# 使用2个不同的嵌入向量和不同的搜索类型定义2个不同的检索者。
retriever_all = db_all.as_retriever(
    search_type="similarity", search_kwargs={"k": 5, "include_metadata": True}
)
retriever_multi_qa = db_multi_qa.as_retriever(
    search_type="mmr", search_kwargs={"k": 5, "include_metadata": True}
)
# 检索者之主将保存两个检索者的输出，并可用作不同类型链上的任何其他检索者。
lotr = MergerRetriever(retrievers=[retriever_all, retriever_multi_qa])
```

## 从合并的检索者结果中删除冗余结果。

```python
# 我们可以使用另一个嵌入向量从两个检索者中删除冗余结果。
# 在不同步骤中使用多个嵌入向量可以帮助减少偏见。
filter = EmbeddingsRedundantFilter(embeddings=filter_embeddings)
pipeline = DocumentCompressorPipeline(transformers=[filter])
compression_retriever = ContextualCompressionRetriever(
    base_compressor=pipeline, base_retriever=lotr
)
```

## 从合并的检索者中挑选代表性文档样本。

```python
# 此过滤器将文档向量分成意义上的簇或“中心”。
# 然后它将选择最接近该中心的文档作为最终结果。
# 默认情况下，结果文档将按簇排序/分组。
filter_ordered_cluster = EmbeddingsClusteringFilter(
    embeddings=filter_embeddings,
    num_clusters=10,
    num_closest=1,
)
# 如果希望最终文档按原始检索者分数排序
# 需要添加“sorted”参数。
filter_ordered_by_retriever = EmbeddingsClusteringFilter(
    embeddings=filter_embeddings,
    num_clusters=10,
    num_closest=1,
    sorted=True,
)
pipeline = DocumentCompressorPipeline(transformers=[filter_ordered_by_retriever])
compression_retriever = ContextualCompressionRetriever(
    base_compressor=pipeline, base_retriever=lotr
)
```

## 重新排序结果以避免性能下降。

无论您的模型架构如何，当包含10个以上的检索文档时，性能会显著下降。

简而言之：当模型必须在长文本中间访问相关信息时，它们往往会忽略提供的文档。

参见：https://arxiv.org/abs//2307.03172

```python
# 您可以使用额外的文档转换器在去除冗余后重新排序文档。
from langchain_community.document_transformers import LongContextReorder
filter = EmbeddingsRedundantFilter(embeddings=filter_embeddings)
reordering = LongContextReorder()
pipeline = DocumentCompressorPipeline(transformers=[filter, reordering])
compression_retriever_reordered = ContextualCompressionRetriever(
    base_compressor=pipeline, base_retriever=lotr
)
```