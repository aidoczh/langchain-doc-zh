# Xata

> [Xata](https://xata.io) 是一个基于 PostgreSQL 的无服务器数据平台。它提供了一个 Python SDK 用于与数据库交互，并提供了一个用户界面用于管理数据。

> Xata 具有原生的向量类型，可以添加到任何表中，并支持相似性搜索。LangChain 直接将向量插入到 Xata 中，并查询给定向量的最近邻居，这样您就可以将所有 LangChain 嵌入与 Xata 集成。

本笔记本将指导您如何将 Xata 用作 VectorStore。

## 设置

### 创建用作向量存储的数据库

在 [Xata 用户界面](https://app.xata.io) 中创建一个新数据库。您可以随意命名，本文档中我们将使用 `langchain`。

创建一个表，同样您可以随意命名，但我们将使用 `vectors`。通过用户界面添加以下列：

* `content` 类型为 "Text"。这用于存储 `Document.pageContent` 的值。

* `embedding` 类型为 "Vector"。使用您计划使用的模型所使用的维度。在本笔记本中，我们使用的是 OpenAI 嵌入，它有 1536 个维度。

* `source` 类型为 "Text"。这是此示例中用作元数据列。

* 您想要用作元数据的任何其他列。它们从 `Document.metadata` 对象中填充。例如，如果在 `Document.metadata` 对象中有一个 `title` 属性，您可以在表中创建一个 `title` 列，它将被填充。

让我们首先安装我们的依赖项：

```python
%pip install --upgrade --quiet  xata langchain-openai tiktoken langchain
```

让我们将 OpenAI 密钥加载到环境中。如果您没有密钥，可以创建一个 OpenAI 帐户，并在此[页面](https://platform.openai.com/account/api-keys)上创建一个密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

同样，我们需要获取 Xata 的环境变量。您可以通过访问您的[帐户设置](https://app.xata.io/settings)来创建一个新的 API 密钥。要找到数据库 URL，请转到您创建的数据库的设置页面。数据库 URL 应该看起来像这样：`https://demo-uni3q8.eu-west-1.xata.sh/db/langchain`。

```python
api_key = getpass.getpass("Xata API key: ")
db_url = input("Xata database URL (copy it from your DB settings):")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.xata import XataVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

### 创建 Xata 向量存储

让我们导入我们的测试数据集：

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

现在创建实际的向量存储，由 Xata 表支持。

```python
vector_store = XataVectorStore.from_documents(
    docs, embeddings, api_key=api_key, db_url=db_url, table_name="vectors"
)
```

运行上述命令后，如果您转到 Xata 用户界面，您应该会看到加载的文档以及它们的嵌入。

要使用已包含向量内容的现有 Xata 表，初始化 XataVectorStore 构造函数：

```python
vector_store = XataVectorStore(
    api_key=api_key, db_url=db_url, embedding=embeddings, table_name="vectors"
)
```

### 相似性搜索

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
found_docs = vector_store.similarity_search(query)
print(found_docs)
```

### 带分数的相似性搜索（向量距离）

```python
query = "总统对 Ketanji Brown Jackson 说了什么"
result = vector_store.similarity_search_with_score(query)
for doc, score in result:
    print(f"document={doc}, score={score}")
```