# 基于 Kinetica Vectorstore 的检索器

[Kinetica](https://www.kinetica.com/) 是一个集成了向量相似性搜索支持的数据库。

它支持以下功能：

- 精确和近似的最近邻搜索

- L2 距离、内积和余弦距离

这个笔记本展示了如何使用基于 Kinetica 向量存储（`Kinetica`）的检索器。

```python
# 请确保在您的工作环境中安装了此连接器。
%pip install gpudb==7.2.0.1
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
```

```python
## 加载环境变量
from dotenv import load_dotenv
load_dotenv()
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import (
    Kinetica,
    KineticaSettings,
)
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
# Kinetica 需要与数据库的连接。
# 这是如何设置连接的方法。
HOST = os.getenv("KINETICA_HOST", "http://127.0.0.1:9191")
USERNAME = os.getenv("KINETICA_USERNAME", "")
PASSWORD = os.getenv("KINETICA_PASSWORD", "")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")
def create_config() -> KineticaSettings:
    return KineticaSettings(host=HOST, username=USERNAME, password=PASSWORD)
```

## 从向量存储创建检索器

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
# Kinetica 模块将尝试使用集合的名称创建一个表。
# 因此，请确保集合名称是唯一的，并且用户有权限创建表。
COLLECTION_NAME = "state_of_the_union_test"
connection = create_config()
db = Kinetica.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    config=connection,
)
# 从向量存储创建检索器
retriever = db.as_retriever(search_kwargs={"k": 2})
```

## 使用检索器进行搜索

```python
result = retriever.get_relevant_documents(
    "总统对 Ketanji Brown Jackson 说了什么"
)
print(docs[0].page_content)
```
