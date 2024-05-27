# Apache Cassandra

本页面提供了使用[Apache Cassandra®](https://cassandra.apache.org/)作为向量存储的快速入门指南。

> [Cassandra](https://cassandra.apache.org/)是一个NoSQL、面向行的、高度可伸缩且高可用的数据库。从5.0版本开始，该数据库具备了[向量搜索功能](https://cassandra.apache.org/doc/trunk/cassandra/vector-search/overview.html)。

_注意：除了访问数据库外，运行完整示例还需要OpenAI API密钥。_

### 设置和一般依赖项

使用集成需要以下Python包。

```python
%pip install --upgrade --quiet "cassio>=0.1.4"
```

_注意：根据您的LangChain设置，您可能需要安装/升级此演示所需的其他依赖项（特别是需要`datasets`、`openai`、`pypdf`和`tiktoken`的最新版本，以及`langchain-community`）。_

```python
import os
from getpass import getpass
from datasets import (
    load_dataset,
)
from langchain_community.document_loaders import PyPDFLoader
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
os.environ["OPENAI_API_KEY"] = getpass("OPENAI_API_KEY = ")
```

```python
embe = OpenAIEmbeddings()
```

## 导入向量存储

```python
from langchain_community.vectorstores import Cassandra
```

## 连接参数

本页面展示的向量存储集成可以与Cassandra以及使用CQL（Cassandra查询语言）协议的其他派生数据库（如Astra DB）一起使用。

> DataStax [Astra DB](https://docs.datastax.com/en/astra-serverless/docs/vector-search/quickstart.html)是构建在Cassandra上的托管无服务器数据库，提供相同的接口和优势。

根据您是连接到Cassandra集群还是通过CQL连接到Astra DB，创建向量存储对象时需要提供不同的参数。

### 连接到Cassandra集群

您首先需要创建一个`cassandra.cluster.Session`对象，如[Cassandra驱动程序文档](https://docs.datastax.com/en/developer/python-driver/latest/api/cassandra/cluster/#module-cassandra.cluster)中所述。细节会有所不同（例如网络设置和身份验证），但可能类似于以下内容：

```python
from cassandra.cluster import Cluster
cluster = Cluster(["127.0.0.1"])
session = cluster.connect()
```

现在您可以将会话与您想要的键空间名称一起设置为全局的CassIO参数：

```python
import cassio
CASSANDRA_KEYSPACE = input("CASSANDRA_KEYSPACE = ")
cassio.init(session=session, keyspace=CASSANDRA_KEYSPACE)
```

现在您可以创建向量存储：

```python
vstore = Cassandra(
    embedding=embe,
    table_name="cassandra_vector_demo",
    # session=None, keyspace=None  # Uncomment on older versions of LangChain
)
```

_注意：您还可以在创建向量存储时直接传递会话和键空间。然而，使用全局的`cassio.init`设置对于如果您的应用程序以多种方式使用Cassandra（例如，用于向量存储、聊天记忆和LLM响应缓存），是很方便的，因为它允许在一个地方集中管理凭据和数据库连接。_

### 通过CQL连接到Astra DB

在这种情况下，您需要使用以下连接参数初始化CassIO：

- 数据库ID，例如`01234567-89ab-cdef-0123-456789abcdef`

- 令牌，例如`AstraCS:6gBhNmsk135....`（必须是“数据库管理员”令牌）

- 可选的键空间名称（如果省略，将使用数据库的默认键空间）

```python
ASTRA_DB_ID = input("ASTRA_DB_ID = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")
desired_keyspace = input("ASTRA_DB_KEYSPACE (optional, can be left empty) = ")
if desired_keyspace:
    ASTRA_DB_KEYSPACE = desired_keyspace
else:
    ASTRA_DB_KEYSPACE = None
```

```python
import cassio
cassio.init(
    database_id=ASTRA_DB_ID,
    token=ASTRA_DB_APPLICATION_TOKEN,
    keyspace=ASTRA_DB_KEYSPACE,
)
```

现在您可以创建向量存储：

```python
vstore = Cassandra(
    embedding=embe,
    table_name="cassandra_vector_demo",
    # session=None, keyspace=None  # Uncomment on older versions of LangChain
)
```

## 加载数据集

将源数据集中的每个条目转换为`Document`，然后将它们写入向量存储：

```python
philo_dataset = load_dataset("datastax/philosopher-quotes")["train"]
docs = []
for entry in philo_dataset:
    metadata = {"author": entry["author"]}
    doc = Document(page_content=entry["quote"], metadata=metadata)
    docs.append(doc)
inserted_ids = vstore.add_documents(docs)
print(f"\nInserted {len(inserted_ids)} documents.")
```

在上述代码中，`metadata` 字典是从源数据创建的，并且是 `Document` 的一部分。

添加一些新条目，这次使用 `add_texts`：

```python
texts = ["我思故我在。", "直面事物本身！"]
metadatas = [{"author": "笛卡尔"}, {"author": "胡塞尔"}]
ids = ["desc_01", "huss_xy"]
inserted_ids_2 = vstore.add_texts(texts=texts, metadatas=metadatas, ids=ids)
print(f"\n插入了 {len(inserted_ids_2)} 个文档。")
```

注意：您可以通过增加这些批量操作的并发级别来加快 `add_texts` 和 `add_documents` 的执行速度。有关更多详细信息，请查看方法的 `batch_size` 参数。

运行搜索

本节演示了元数据过滤和获取相似度分数：

```python
results = vstore.similarity_search("我们的生活是我们自己创造的", k=3)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```

```python
results_filtered = vstore.similarity_search(
    "我们的生活是我们自己创造的",
    k=3,
    filter={"author": "柏拉图"},
)
for res in results_filtered:
    print(f"* {res.page_content} [{res.metadata}]")
```

```python
results = vstore.similarity_search_with_score("我们的生活是我们自己创造的", k=3)
for res, score in results:
    print(f"* [相似度={score:3f}] {res.page_content} [{res.metadata}]")
```

MMR（最大边际相关性）搜索

```python
results = vstore.max_marginal_relevance_search(
    "我们的生活是我们自己创造的",
    k=3,
    filter={"author": "亚里士多德"},
)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```

删除存储的文档

```python
delete_1 = vstore.delete(inserted_ids[:3])
print(f"all_succeed={delete_1}")  # True，所有文档都被删除
delete_2 = vstore.delete(inserted_ids[2:5])
print(f"some_succeeds={delete_2}")  # True，尽管有些 ID 已经不存在
```

一个简单的 RAG 链

下面的代码将实现一个简单的 RAG 流程：

- 下载一个示例 PDF 文件并将其加载到存储中；

- 使用 LCEL（LangChain 表达式语言）创建一个 RAG 链，其中心是向量存储；

- 运行问答链。

```python
!curl -L \
    "https://github.com/awesome-astra/datasets/blob/main/demo-resources/what-is-philosophy/what-is-philosophy.pdf?raw=true" \
    -o "what-is-philosophy.pdf"
```

```python
pdf_loader = PyPDFLoader("what-is-philosophy.pdf")
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=64)
docs_from_pdf = pdf_loader.load_and_split(text_splitter=splitter)
print(f"从 PDF 中提取的文档数：{len(docs_from_pdf)}。")
inserted_ids_from_pdf = vstore.add_documents(docs_from_pdf)
print(f"插入了 {len(inserted_ids_from_pdf)} 个文档。")
```

```python
retriever = vstore.as_retriever(search_kwargs={"k": 3})
philo_template = """
您是一位哲学家，从过去的伟大思想家那里汲取灵感，为用户的问题提供深思熟虑的答案。将提供的上下文作为答案的基础，不要编造新的推理路径，只需混合和匹配您所获得的信息。您的答案必须简明扼要，不要回答与哲学无关的其他主题。
上下文：
{context}
问题：{question}
您的答案："""
philo_prompt = ChatPromptTemplate.from_template(philo_template)
llm = ChatOpenAI()
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | philo_prompt
    | llm
    | StrOutputParser()
)
```

```python
chain.invoke("Russel 如何阐述 Peirce 的安全毯理论？")
```

有关更多信息，请查看使用 LangChain `Cassandra` 向量存储的完整 RAG 模板，请访问 [此处](https://github.com/langchain-ai/langchain/tree/master/templates/cassandra-entomology-rag)。

清理

以下代码从 CassIO 中检索 `Session` 对象，并使用 CQL `DROP TABLE` 语句运行它：

（您将丢失其中存储的数据。）

```python
cassio.config.resolve_session().execute(
    f"DROP TABLE {cassio.config.resolve_keyspace()}.cassandra_vector_demo;"
)
```

了解更多

有关更多信息、扩展的快速入门和其他使用示例，请访问 [CassIO 文档](https://cassio.org/frameworks/langchain/about/) 以了解如何使用 LangChain `Cassandra` 向量存储。

归属声明

> Apache Cassandra、Cassandra 和 Apache 是 [Apache Software Foundation](http://www.apache.org/) 在美国和/或其他国家的注册商标或商标。