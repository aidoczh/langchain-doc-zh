# SingleStoreDB

[SingleStoreDB](https://singlestore.com/) 是一个高性能的分布式 SQL 数据库，支持部署在[云端](https://www.singlestore.com/cloud/)和本地。它提供向量存储和向量函数，包括[点积](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/dot_product.html)和[欧几里得距离](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/euclidean_distance.html)，从而支持需要文本相似度匹配的人工智能应用程序。

这个笔记本展示了如何使用一个使用 `SingleStoreDB` 的检索器。

```python
# 通过 singlestoredb Python 连接器方便地建立与数据库的连接。
# 请确保在您的工作环境中安装了这个连接器。
%pip install --upgrade --quiet  singlestoredb
```

## 从向量存储创建检索器

```python
import getpass
import os
# 我们想使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import SingleStoreDB
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
# 设置连接 URL 作为环境变量
os.environ["SINGLESTOREDB_URL"] = "root:pass@localhost:3306/db"
# 将文档加载到存储中
docsearch = SingleStoreDB.from_documents(
    docs,
    embeddings,
    table_name="notebook",  # 使用自定义名称的表
)
# 从向量存储创建检索器
retriever = docsearch.as_retriever(search_kwargs={"k": 2})
```

## 使用检索器进行搜索

```python
result = retriever.invoke("总统关于 Ketanji Brown Jackson 说了什么")
print(docs[0].page_content)
```