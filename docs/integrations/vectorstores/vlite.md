# VLite

VLite 是一个简单且速度极快的向量数据库，允许您使用嵌入来语义化地存储和检索数据。VLite 使用 numpy 制作，是一个轻量级的、内置电池的数据库，可用于在项目中实现 RAG、相似性搜索和嵌入。

## 安装

要在 LangChain 中使用 VLite，您需要安装 `vlite` 包：

```bash
!pip install vlite
```

## 导入 VLite

```python
from langchain.vectorstores import VLite
```

## 基本示例

在这个基本示例中，我们加载一个文本文档，并将其存储在 VLite 向量数据库中。然后，我们执行相似性搜索，以根据查询检索相关文档。

VLite 会为您处理文本的分块和嵌入，并且您可以通过预分块文本和/或将这些分块嵌入到 VLite 数据库中来更改这些参数。

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter
# 加载文档并将其分成块
loader = TextLoader("path/to/document.txt")
documents = loader.load()
# 创建一个 VLite 实例
vlite = VLite(collection="my_collection")
# 将文档添加到 VLite 向量数据库
vlite.add_documents(documents)
# 执行相似性搜索
query = "文档的主题是什么？"
docs = vlite.similarity_search(query)
# 打印最相关的文档
print(docs[0].page_content)
```

## 添加文本和文档

您可以使用 `add_texts` 和 `add_documents` 方法将文本或文档添加到 VLite 向量数据库中。

```python
# 将文本添加到 VLite 向量数据库
texts = ["这是第一段文本。", "这是第二段文本。"]
vlite.add_texts(texts)
# 将文档添加到 VLite 向量数据库
documents = [Document(page_content="这是一个文档。", metadata={"source": "example.txt"})]
vlite.add_documents(documents)
```

## 相似性搜索

VLite 提供了执行存储文档的相似性搜索的方法。

```python
# 执行相似性搜索
query = "文档的主题是什么？"
docs = vlite.similarity_search(query, k=3)
# 带有分数的相似性搜索
docs_with_scores = vlite.similarity_search_with_score(query, k=3)
```

## 最大边际相关性搜索

VLite 还支持最大边际相关性（MMR）搜索，该搜索优化了查询与检索文档之间的相似性和多样性。

```python
# 执行 MMR 搜索
docs = vlite.max_marginal_relevance_search(query, k=3)
```

## 更新和删除文档

您可以使用 `update_document` 和 `delete` 方法在 VLite 向量数据库中更新或删除文档。

```python
# 更新文档
document_id = "doc_id_1"
updated_document = Document(page_content="更新后的内容", metadata={"source": "updated.txt"})
vlite.update_document(document_id, updated_document)
# 删除文档
document_ids = ["doc_id_1", "doc_id_2"]
vlite.delete(document_ids)
```

## 检索文档

您可以使用 `get` 方法根据其 ID 或元数据从 VLite 向量数据库中检索文档。

```python
# 根据 ID 检索文档
document_ids = ["doc_id_1", "doc_id_2"]
docs = vlite.get(ids=document_ids)
# 根据元数据检索文档
metadata_filter = {"source": "example.txt"}
docs = vlite.get(where=metadata_filter)
```

## 创建 VLite 实例

您可以使用各种方法创建 VLite 实例：

```python
# 从文本创建 VLite 实例
vlite = VLite.from_texts(texts)
# 从文档创建 VLite 实例
vlite = VLite.from_documents(documents)
# 从现有索引创建 VLite 实例
vlite = VLite.from_existing_index(collection="existing_collection")
```

## 附加功能

VLite 提供了用于管理向量数据库的附加功能：

```python
from langchain.vectorstores import VLite
vlite = VLite(collection="my_collection")
# 获取集合中的项目数
count = vlite.count()
# 保存集合
vlite.save()
# 清空集合
vlite.clear()
# 获取集合信息
vlite.info()
# 转储集合数据
data = vlite.dump()
```