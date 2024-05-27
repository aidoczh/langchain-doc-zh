# 亚马逊文档数据库

[亚马逊文档数据库（与 MongoDB 兼容）](https://docs.aws.amazon.com/documentdb/) 可以轻松在云中设置、操作和扩展与 MongoDB 兼容的数据库。

使用亚马逊文档数据库，您可以运行相同的应用程序代码，并使用与 MongoDB 相同的驱动程序和工具。

亚马逊文档数据库的向量搜索结合了基于 JSON 的文档数据库的灵活性和丰富的查询能力，以及向量搜索的强大功能。

这个笔记本向您展示了如何使用[亚马逊文档数据库向量搜索](https://docs.aws.amazon.com/documentdb/latest/developerguide/vector-search.html)来存储集合中的文档，创建索引，并使用近似最近邻算法进行向量搜索查询，比如 "cosine"、"euclidean" 和 "dotProduct"。默认情况下，DocumentDB 创建分层可导航小世界（HNSW）索引。要了解其他支持的向量索引类型，请参阅上面链接的文档。

要使用 DocumentDB，您必须首先部署一个集群。有关更多详细信息，请参阅[开发人员指南](https://docs.aws.amazon.com/documentdb/latest/developerguide/what-is.html)。

[立即注册](https://aws.amazon.com/free/)免费开始使用。

```python
!pip install pymongo
```

```python
import getpass
# DocumentDB 连接字符串
# 例如："mongodb://{username}:{pass}@{cluster_endpoint}:{port}/?{params}"
CONNECTION_STRING = getpass.getpass("DocumentDB Cluster URI:")
INDEX_NAME = "izzy-test-index"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要设置我们的 OpenAI 环境变量。

```python
import getpass
import os
# 设置 OpenAI 环境变量
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
os.environ["OPENAI_EMBEDDINGS_DEPLOYMENT"] = (
    "smart-agent-embedding-ada"  # 嵌入模型的部署名称
)
os.environ["OPENAI_EMBEDDINGS_MODEL_NAME"] = "text-embedding-ada-002"  # 模型名称
```

现在，我们将加载文档到集合中，创建索引，然后对索引执行查询。如果您对某些参数有疑问，请参阅[文档](https://docs.aws.amazon.com/documentdb/latest/developerguide/vector-search.html)。

```python
from langchain.vectorstores.documentdb import (
    DocumentDBSimilarityType,
    DocumentDBVectorSearch,
)
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
SOURCE_FILE_NAME = "../../how_to/state_of_the_union.txt"
loader = TextLoader(SOURCE_FILE_NAME)
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
# OpenAI 设置
model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")
openai_embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    deployment=model_deployment, model=model_name
)
```

```python
from pymongo import MongoClient
INDEX_NAME = "izzy-test-index-2"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")
client: MongoClient = MongoClient(CONNECTION_STRING)
collection = client[DB_NAME][COLLECTION_NAME]
model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")
vectorstore = DocumentDBVectorSearch.from_documents(
    documents=docs,
    embedding=openai_embeddings,
    collection=collection,
    index_name=INDEX_NAME,
)
# 模型使用的维度数
dimensions = 1536
# 指定相似性算法，有效选项包括：
#   cosine (COS), euclidean (EUC), dotProduct (DOT)
similarity_algorithm = DocumentDBSimilarityType.COS
vectorstore.create_index(dimensions, similarity_algorithm)
```

```output
{ 'createdCollectionAutomatically' : false,
   'numIndexesBefore' : 1,
   'numIndexesAfter' : 2,
   'ok' : 1,
   'operationTime' : Timestamp(1703656982, 1)}
```

```python
# 对查询的嵌入和文档的嵌入执行相似性搜索
query = "What did the President say about Ketanji Brown Jackson"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

一旦文档被加载并且索引被创建，你现在可以直接实例化向量存储并对索引运行查询了。

```python
vectorstore = DocumentDBVectorSearch.from_connection_string(
    connection_string=CONNECTION_STRING,
    namespace=NAMESPACE,
    embedding=openai_embeddings,
    index_name=INDEX_NAME,
)
# 对已摄入的文档执行查询的相似性搜索
query = "总统对凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）有何评价"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。还有，顺便通过《披露法案》，这样美国人就能知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。
而我在4天前就做到了，当我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。
```

```python
# 对已摄入的文档执行查询的相似性搜索
query = "总统分享了关于美国经济的哪些统计数据"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
与前届政府通过的2万亿美元的减税不同，该减税惠及美国顶层1%的人，而《美国复苏计划》帮助了劳动人民，没有让任何人掉队。
而且它奏效了。它创造了就业机会。大量的就业机会。
事实上——我们的经济仅去年就创造了超过650万个新工作岗位，这是美国历史上有史以来创造的最多工作岗位。
我们的经济去年增长了5.7%，这是近40年来最强劲的增长，这是为一个长期未能造福这个国家劳动人民的经济带来根本性变革的第一步。
在过去的40年里，我们被告知，如果我们给予最顶层的人减税，好处会向下渗透到其他人。
但是这种“向下渗透”的理论导致了经济增长的减弱、工资的下降、赤字的增加，以及近一个世纪以来最广泛的顶层和其他人之间的差距。
```

## 问答

```python
qa_retriever = vectorstore.as_retriever(
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
docs = qa({"query": "gpt-4 计算要求"})
print(docs["result"])
print(docs["source_documents"])
```