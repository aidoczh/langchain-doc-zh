# DingoDB

[DingoDB](https://dingodb.readthedocs.io/en/latest/) 是一个分布式多模式向量数据库，结合了数据湖和向量数据库的特点，可以存储任何类型和大小的数据（键-值、PDF、音频、视频等）。它具有实时低延迟处理能力，实现快速洞察和响应，并能高效地进行即时分析和处理多模态数据。

这个笔记展示了如何使用与 DingoDB 向量数据库相关的功能。

要运行此笔记，您应该已经启动并运行了一个 [DingoDB 实例](https://github.com/dingodb/dingo-deploy/blob/main/README.md)。

```python
%pip install --upgrade --quiet  dingodb
# 或者安装最新版本:
%pip install --upgrade --quiet  git+https://git@github.com/dingodb/pydingo.git
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key:········
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Dingo
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
from dingodb import DingoDB
index_name = "langchain_demo"
dingo_client = DingoDB(user="", password="", host=["127.0.0.1:13000"])
# 首先，检查我们的索引是否已经存在。如果不存在，我们将创建它
if (
    index_name not in dingo_client.get_index()
    and index_name.upper() not in dingo_client.get_index()
):
    # 我们创建一个新的索引，根据您自己的情况进行修改
    dingo_client.create_index(
        index_name=index_name, dimension=1536, metric_type="cosine", auto_id=False
    )
# OpenAI 嵌入模型 `text-embedding-ada-002` 使用 1536 维度
docsearch = Dingo.from_documents(
    docs, embeddings, client=dingo_client, index_name=index_name
)
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Dingo
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
query = "总统对 Ketanji Brown Jackson 有什么评论"
docs = docsearch.similarity_search(query)
```

```python
print(docs[0].page_content)
```

### 向现有索引添加更多文本

可以使用 `add_texts` 函数将更多文本嵌入并更新到现有的 Dingo 索引中。

```python
vectorstore = Dingo(embeddings, "text", client=dingo_client, index_name=index_name)
vectorstore.add_texts(["更多文本！"])
```

### 最大边际相关搜索

除了在 retriever 对象中使用相似性搜索之外，您还可以使用 `mmr` 作为检索器。

```python
retriever = docsearch.as_retriever(search_type="mmr")
matched_docs = retriever.invoke(query)
for i, d in enumerate(matched_docs):
    print(f"\n## 文档 {i}\n")
    print(d.page_content)
```

或者直接使用 `max_marginal_relevance_search`：

```python
found_docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10)
for i, doc in enumerate(found_docs):
    print(f"{i + 1}.", doc.page_content, "\n")
```