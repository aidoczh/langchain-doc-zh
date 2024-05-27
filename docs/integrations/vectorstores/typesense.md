# Typesense

[Typesense](https://typesense.org) 是一款开源的内存搜索引擎，您可以选择[自行托管](https://typesense.org/docs/guide/install-typesense#option-2-local-machine-self-hosting)，也可以在[Typesense 云](https://cloud.typesense.org/)上运行。

Typesense 专注于性能，通过将整个索引存储在 RAM 中（同时备份在磁盘上），并通过简化可用选项和设置良好的默认值，提供开箱即用的开发者体验。

它还允许您将基于属性的过滤与向量查询结合起来，以获取最相关的文档。

这篇笔记将向您展示如何将 Typesense 用作您的 VectorStore。

首先让我们安装我们的依赖项：

```python
%pip install --upgrade --quiet  typesense openapi-schema-pydantic langchain-openai tiktoken
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Typesense
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

让我们导入我们的测试数据集：

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

```python
docsearch = Typesense.from_documents(
    docs,
    embeddings,
    typesense_client_params={
        "host": "localhost",  # Use xxx.a1.typesense.net for Typesense Cloud
        "port": "8108",  # Use 443 for Typesense Cloud
        "protocol": "http",  # Use https for Typesense Cloud
        "typesense_api_key": "xyz",
        "typesense_collection_name": "lang-chain",
    },
)
```

## 相似度搜索

```python
query = "总统对Ketanji Brown Jackson有何看法"
found_docs = docsearch.similarity_search(query)
```

```python
print(found_docs[0].page_content)
```

## Typesense 作为检索器

Typesense，像其他向量存储一样，是 LangChain 检索器，使用余弦相似度。

```python
retriever = docsearch.as_retriever()
retriever
```

```python
query = "总统对Ketanji Brown Jackson有何看法"
retriever.invoke(query)[0]
```