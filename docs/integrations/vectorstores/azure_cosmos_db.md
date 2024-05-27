# Azure Cosmos DB

本笔记本向您展示如何利用这个集成的[向量数据库](https://learn.microsoft.com/en-us/azure/cosmos-db/vector-database)来存储集合中的文档，创建索引，并使用近似最近邻算法（如 COS（余弦距离）、L2（欧几里德距离）和IP（内积））执行向量搜索查询，以定位接近查询向量的文档。

Azure Cosmos DB是支持 OpenAI 的 ChatGPT 服务的数据库。它提供单位毫秒的响应时间，自动和即时的可扩展性，以及在任何规模下的速度保证。

Azure Cosmos DB for MongoDB vCore(https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/) 为开发人员提供了一个完全托管的与 MongoDB 兼容的数据库服务，用于构建具有熟悉架构的现代应用程序。您可以应用您的 MongoDB 经验，并继续使用您喜爱的 MongoDB 驱动程序、SDK 和工具，通过将您的应用程序指向 MongoDB vCore 帐户的连接字符串来实现。

[立即注册](https://azure.microsoft.com/en-us/free/)，获得终身免费访问，开始使用吧。

```python
%pip install --upgrade --quiet  pymongo
```

```output
[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
```

```python
import os
CONNECTION_STRING = "YOUR_CONNECTION_STRING"
INDEX_NAME = "izzy-test-index"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要设置 Azure OpenAI API 密钥以及其他环境变量。

```python
# 设置 OpenAI 环境变量
os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_VERSION"] = "2023-05-15"
os.environ["OPENAI_API_BASE"] = (
    "YOUR_OPEN_AI_ENDPOINT"  # https://example.openai.azure.com/
)
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
os.environ["OPENAI_EMBEDDINGS_DEPLOYMENT"] = (
    "smart-agent-embedding-ada"  # 嵌入模型的部署名称
)
os.environ["OPENAI_EMBEDDINGS_MODEL_NAME"] = "text-embedding-ada-002"  # 模型名称
```

现在，我们需要将文档加载到集合中，创建索引，然后针对索引运行我们的查询以检索匹配项。

如果您对某些参数有疑问，请参阅[文档](https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search)。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.azure_cosmos_db import (
    AzureCosmosDBVectorSearch,
    CosmosDBSimilarityType,
    CosmosDBVectorSearchType,
)
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
    deployment=model_deployment, model=model_name, chunk_size=1
)
```

```python
from pymongo import MongoClient
client: MongoClient = MongoClient(CONNECTION_STRING)
collection = client[DB_NAME][COLLECTION_NAME]
model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")
vectorstore = AzureCosmosDBVectorSearch.from_documents(
    docs,
    openai_embeddings,
    collection=collection,
    index_name=INDEX_NAME,
)
# 在此处详细了解这些变量。https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search
num_lists = 100
dimensions = 1536
similarity_algorithm = CosmosDBSimilarityType.COS
kind = CosmosDBVectorSearchType.VECTOR_IVF
m = 16
ef_construction = 64
ef_search = 40
score_threshold = 0.1
vectorstore.create_index(
    num_lists, dimensions, similarity_algorithm, kind, m, ef_construction
)
```

```output
{'raw': {'defaultShard': {'numIndexesBefore': 1,
   'numIndexesAfter': 2,
   'createdCollectionAutomatically': False,
   'ok': 1}},
 'ok': 1}
```

```python
# 对查询的嵌入和文档的嵌入执行相似性搜索
query = "What did the president say about Ketanji Brown Jackson"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。 
```

一旦文档加载完成并创建了索引，您现在可以直接实例化向量存储并针对索引运行查询。

```python
vectorstore = AzureCosmosDBVectorSearch.from_connection_string(
    CONNECTION_STRING, NAMESPACE, openai_embeddings, index_name=INDEX_NAME
)
# 对已摄取的文档执行查询的相似性搜索
query = "总统对凯坦吉·布朗·杰克逊有何评价"
docs = vectorstore.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。
```

```python
vectorstore = AzureCosmosDBVectorSearch(
    collection, openai_embeddings, index_name=INDEX_NAME
)
# 对已摄取的文档执行查询的相似性搜索
query = "总统对凯坦吉·布朗·杰克逊有何评价"
docs = vectorstore.similarity_search(query)
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《揭露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。
```