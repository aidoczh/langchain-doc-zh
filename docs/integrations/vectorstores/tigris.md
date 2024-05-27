# Tigris

> [Tigris](https://tigrisdata.com) 是一个开源的无服务器 NoSQL 数据库和搜索平台，旨在简化构建高性能向量搜索应用程序。

> `Tigris` 消除了管理、操作和同步多个工具的基础设施复杂性，使您能够专注于构建出色的应用程序。

本笔记将指导您如何将 Tigris 用作您的 VectorStore。

**先决条件**

1. 一个 OpenAI 账户。您可以在[这里](https://platform.openai.com/)注册账户。

2. [注册免费的 Tigris 账户](https://console.preview.tigrisdata.cloud)。注册 Tigris 账户后，创建一个名为 `vectordemo` 的新项目。然后，记下您在项目的**应用密钥**部分获取的 *Uri*、**clientId** 和 **clientSecret**。

让我们首先安装我们的依赖项：

```python
%pip install --upgrade --quiet  tigrisdb openapi-schema-pydantic langchain-openai tiktoken
```

我们将在环境中加载 `OpenAI` api 密钥和 `Tigris` 凭据：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
os.environ["TIGRIS_PROJECT"] = getpass.getpass("Tigris Project Name:")
os.environ["TIGRIS_CLIENT_ID"] = getpass.getpass("Tigris Client Id:")
os.environ["TIGRIS_CLIENT_SECRET"] = getpass.getpass("Tigris Client Secret:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Tigris
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

### 初始化 Tigris 向量存储

让我们导入我们的测试数据集：

```python
loader = TextLoader("../../../state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
vector_store = Tigris.from_documents(docs, embeddings, index_name="my_embeddings")
```

### 相似性搜索

```python
query = "总统对Ketanji Brown Jackson有何看法"
found_docs = vector_store.similarity_search(query)
print(found_docs)
```

### 带分数的相似性搜索（向量距离）

```python
query = "总统对Ketanji Brown Jackson有何看法"
result = vector_store.similarity_search_with_score(query)
for doc, score in result:
    print(f"document={doc}, score={score}")
```