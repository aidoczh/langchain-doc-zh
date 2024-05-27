# Viking DB

[viking DB](https://www.volcengine.com/docs/6459/1163946) 是一个存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大规模嵌入向量的数据库。

这个笔记本展示了如何使用与 VikingDB 向量数据库相关的功能。

要运行，您应该已经启动并运行了一个 [viking DB 实例](https://www.volcengine.com/docs/6459/1165058)。

```python
!pip install --upgrade volcengine
```

我们想要使用 VikingDBEmbeddings，因此我们需要获取 VikingDB API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.vikingdb import VikingDB, VikingDBConfig
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
loader = TextLoader("./test.txt")
documents = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=10, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
db = VikingDB.from_documents(
    docs,
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    drop_old=True,
)
```

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```

```python
docs[0].page_content
```

### 使用 Viking DB 集合对数据进行分隔

您可以将不同的不相关文档存储在同一个 viking DB 实例中的不同集合中，以保持上下文。

以下是如何创建一个新的集合。

```python
db = VikingDB.from_documents(
    docs,
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    collection_name="collection_1",
    drop_old=True,
)
```

以下是如何检索已存储的集合。

```python
db = VikingDB.from_documents(
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    collection_name="collection_1",
)
```

检索后，您可以像往常一样进行查询。