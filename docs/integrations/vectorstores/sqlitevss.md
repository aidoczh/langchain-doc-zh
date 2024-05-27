# SQLite-VSS

[SQLite-VSS](https://alexgarcia.xyz/sqlite-vss/) 是一个为向量搜索而设计的 `SQLite` 扩展，强调本地优先操作，并且可以轻松集成到应用程序中，无需外部服务器。利用 `Faiss` 库，它提供了高效的相似性搜索和聚类能力。

这个笔记本展示了如何使用 `SQLiteVSS` 向量数据库。

```python
# 你需要将 sqlite-vss 安装为依赖项。
%pip install --upgrade --quiet  sqlite-vss
```

## 快速开始

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.sentence_transformer import (
    SentenceTransformerEmbeddings,
)
from langchain_community.vectorstores import SQLiteVSS
from langchain_text_splitters import CharacterTextSplitter
# 加载文档并将其分割成块
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
# 将其分割成块
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
texts = [doc.page_content for doc in docs]
# 创建开源嵌入函数
embedding_function = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
# 将其加载到名为 state_union 的 sqlite-vss 表中。
# db_file 参数是你想要作为 sqlite 数据库的文件名。
db = SQLiteVSS.from_texts(
    texts=texts,
    embedding=embedding_function,
    table="state_union",
    db_file="/tmp/vss.db",
)
# 查询
query = "总统对 Ketanji Brown Jackson 有什么看法"
data = db.similarity_search(query)
# 打印结果
data[0].page_content
```

```output
'今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。\n\n总统的最重要的宪法责任之一就是提名某人担任美国最高法院的法官。\n\n而我在4天前就做到了，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶法官的卓越传统。'
```

## 使用现有的 SQLite 连接

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.sentence_transformer import (
    SentenceTransformerEmbeddings,
)
from langchain_community.vectorstores import SQLiteVSS
from langchain_text_splitters import CharacterTextSplitter
# 加载文档并将其分割成块
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
# 将其分割成块
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
texts = [doc.page_content for doc in docs]
# 创建开源嵌入函数
embedding_function = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
connection = SQLiteVSS.create_connection(db_file="/tmp/vss.db")
db1 = SQLiteVSS(
    table="state_union", embedding=embedding_function, connection=connection
)
db1.add_texts(["Ketanji Brown Jackson is awesome"])
# 再次查询
query = "总统对 Ketanji Brown Jackson 有什么看法"
data = db1.similarity_search(query)
# 打印结果
data[0].page_content
```

```output
'Ketanji Brown Jackson is awesome'
```

```python
# 清理
import os
os.remove("/tmp/vss.db")
```