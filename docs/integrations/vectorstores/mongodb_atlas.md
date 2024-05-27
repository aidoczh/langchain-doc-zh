# MongoDB Atlas

[MongoDB Atlas](https://www.mongodb.com/docs/atlas/) 是一款在 AWS、Azure 和 GCP 上提供的全托管云数据库。现在它支持在 MongoDB 文档数据上进行原生向量搜索。

本文档展示了如何使用 [MongoDB Atlas Vector Search](https://www.mongodb.com/products/platform/atlas-vector-search) 将嵌入存储在 MongoDB 文档中，创建一个向量搜索索引，并使用近似最近邻算法（`Hierarchical Navigable Small Worlds`）执行 KNN 搜索。它使用了 [$vectorSearch MQL Stage](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/)。

要使用 MongoDB Atlas，您必须首先部署一个集群。我们有免费永久集群层级可用。要开始，请访问 Atlas 这里：[快速入门](https://www.mongodb.com/docs/atlas/getting-started/)。

> 注意：

>

> - 更多文档可以在 [LangChain-MongoDB 网站](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/) 找到。

> - 此功能已经普遍可用，并且可以用于生产部署。

> - langchain 版本 0.0.305 ([发布说明](https://github.com/langchain-ai/langchain/releases/tag/v0.0.305)) 引入了对 $vectorSearch MQL stage 的支持，该支持可在 MongoDB Atlas 6.0.11 和 7.0.2 中使用。使用较早版本的 MongoDB Atlas 的用户需要将其 LangChain 版本固定为 <=0.0.304。

在本文档中，我们将演示如何使用 MongoDB Atlas、OpenAI 和 Langchain 执行 `检索增强生成`（RAG）。我们将执行相似性搜索、带有元数据预过滤的相似性搜索，并针对于 2023 年 3 月发布的 [GPT 4 技术报告](https://arxiv.org/pdf/2303.08774.pdf) 进行 PDF 文档的问答，因此不属于 OpenAI 的大型语言模型（LLM）的参数化记忆，该模型的知识截止日期为 2021 年 9 月。

我们希望使用 `OpenAIEmbeddings`，因此我们需要设置我们的 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

现在我们将设置 MongoDB Atlas 集群的环境变量。

```python
%pip install --upgrade --quiet  langchain pypdf pymongo langchain-openai tiktoken
```

```python
import getpass
MONGODB_ATLAS_CLUSTER_URI = getpass.getpass("MongoDB Atlas Cluster URI:")
```

```python
from pymongo import MongoClient
# 初始化 MongoDB Python 客户端
client = MongoClient(MONGODB_ATLAS_CLUSTER_URI)
DB_NAME = "langchain_db"
COLLECTION_NAME = "test"
ATLAS_VECTOR_SEARCH_INDEX_NAME = "index_name"
MONGODB_COLLECTION = client[DB_NAME][COLLECTION_NAME]
```

## 创建向量搜索索引

现在，让我们在您的集群上创建一个向量搜索索引。更详细的步骤可以在 [为 LangChain 创建向量搜索索引](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/#create-the-atlas-vector-search-index) 部分找到。

在下面的示例中，`embedding` 是包含嵌入向量的字段的名称。请参阅 [文档](https://www.mongodb.com/docs/atlas/atlas-vector-search/create-index/) 以获取有关如何定义 Atlas Vector Search 索引的更多详细信息。

您可以将索引命名为 `{ATLAS_VECTOR_SEARCH_INDEX_NAME}`，并在命名空间 `{DB_NAME}.{COLLECTION_NAME}` 上创建索引。最后，在 MongoDB Atlas 的 JSON 编辑器中写入以下定义：

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    }
  ]
}
```

# 插入数据

```python
from langchain_community.document_loaders import PyPDFLoader
# 加载 PDF
loader = PyPDFLoader("https://arxiv.org/pdf/2303.08774.pdf")
data = loader.load()
```

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
docs = text_splitter.split_documents(data)
```

```python
print(docs[0])
```

```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings
# 使用它们的嵌入将文档插入 MongoDB Atlas
vector_search = MongoDBAtlasVectorSearch.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings(disallowed_special=()),
    collection=MONGODB_COLLECTION,
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```

```python
# 在查询的嵌入和文档的嵌入之间执行相似性搜索
query = "What were the compute requirements for training GPT 4"
results = vector_search.similarity_search(query)
print(results[0].page_content)
```

# 查询数据

我们还可以直接实例化向量存储并执行查询，如下所示：

```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings
vector_search = MongoDBAtlasVectorSearch.from_connection_string(
    MONGODB_ATLAS_CLUSTER_URI,
    DB_NAME + "." + COLLECTION_NAME,
    OpenAIEmbeddings(disallowed_special=()),
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```

## 使用相似性搜索进行预过滤

Atlas Vector Search 支持使用 MQL 运算符进行过滤的预过滤。以下是对相同数据进行的索引和查询示例，允许您在“page”字段上进行元数据过滤。您可以使用定义的过滤器更新现有索引，并使用向量搜索进行预过滤。

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    },
    {
      "type": "filter",
      "path": "page"
    }
  ]
}
```

```python
query = "训练 GPT-4 的计算需求是什么"
results = vector_search.similarity_search_with_score(
    query=query, k=5, pre_filter={"page": {"$eq": 1}}
)
# 显示结果
for result in results:
    print(result)
```

## 带分数的相似性搜索

```python
query = "训练 GPT-4 的计算需求是什么"
results = vector_search.similarity_search_with_score(
    query=query,
    k=5,
)
# 显示结果
for result in results:
    print(result)
```

## 问答

```python
qa_retriever = vector_search.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 25},
)
```

```python
from langchain_core.prompts import PromptTemplate
prompt_template = """使用以下上下文片段来回答最后的问题。如果你不知道答案，就说你不知道，不要编造答案。
{context}
问题：{question}
"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
```

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAI
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=qa_retriever,
    return_source_documents=True,
    chain_type_kwargs={"prompt": PROMPT},
)
docs = qa({"query": "gpt-4 计算需求"})
print(docs["result"])
print(docs["source_documents"])
```

GPT-4 需要的计算量比之前的 GPT 模型大得多。在从 OpenAI 内部代码库衍生的数据集上，GPT-4 需要 100p（petaflops）的计算量才能达到最低损失，而较小的模型需要 1-10n（nanoflops）。