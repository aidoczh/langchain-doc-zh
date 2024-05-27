# DuckDB

本笔记展示了如何将 `DuckDB` 用作向量存储。

```python
! pip install duckdb
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
```

```python
from langchain.vectorstores import DuckDB
from langchain_openai import OpenAIEmbeddings
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
documents = CharacterTextSplitter().split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
docsearch = DuckDB.from_documents(documents, embeddings)
query = "总统对Ketanji Brown Jackson有什么评论"
docs = docsearch.similarity_search(query)
```

```python
print(docs[0].page_content)
```