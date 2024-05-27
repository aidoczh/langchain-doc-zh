# 松鼠混合搜索

>[松鼠](https://docs.pinecone.io/docs/overview) 是一个功能广泛的向量数据库。

本文介绍了如何使用一个底层使用松鼠和混合搜索的检索器。

该检索器的逻辑取自[此文档](https://docs.pinecone.io/docs/hybrid-search)。

要使用松鼠，您必须拥有一个 API 密钥和一个环境。以下是[安装说明](https://docs.pinecone.io/docs/quickstart)。

```python
%pip install --upgrade --quiet  pinecone-client pinecone-text pinecone-notebooks
```

```python
# 连接到松鼠并获取 API 密钥。
from pinecone_notebooks.colab import Authenticate
Authenticate()
import os
api_key = os.environ["PINECONE_API_KEY"]
```

```python
from langchain_community.retrievers import (
    PineconeHybridSearchRetriever,
)
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
```

## 设置松鼠

您只需要执行这一部分一次。

```python
import os
from pinecone import Pinecone, ServerlessSpec
index_name = "langchain-pinecone-hybrid-search"
# 初始化松鼠客户端
pc = Pinecone(api_key=api_key)
# 创建索引
if index_name not in pc.list_indexes().names():
    pc.create_index(
        name=index_name,
        dimension=1536,  # 密集模型的维度
        metric="dotproduct",  # 仅支持 dotproduct 的稀疏值
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
    )
```

```output
WhoAmIResponse(username='load', user_label='label', projectname='load-test')
```

现在索引已经创建，我们可以使用它。

```python
index = pc.Index(index_name)
```

## 获取嵌入和稀疏编码器

嵌入用于密集向量，分词器用于稀疏向量。

```python
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

要将文本编码为稀疏值，您可以选择 SPLADE 或 BM25。对于领域外任务，我们建议使用 BM25。

有关稀疏编码器的更多信息，您可以查看 pinecone-text 库的[文档](https://pinecone-io.github.io/pinecone-text/pinecone_text.html)。

```python
from pinecone_text.sparse import BM25Encoder
# 或者从 pinecone_text.sparse 导入 SpladeEncoder，如果您希望使用 SPLADE
# 使用默认的 tf-idf 值
bm25_encoder = BM25Encoder().default()
```

上述代码使用了默认的 tf-idf 值。强烈建议将 tf-idf 值拟合到您自己的语料库中。您可以按照以下方式进行操作：

```python
corpus = ["foo", "bar", "world", "hello"]
# 在您的语料库上拟合 tf-idf 值
bm25_encoder.fit(corpus)
# 将值存储到一个 json 文件中
bm25_encoder.dump("bm25_values.json")
# 加载到您的 BM25Encoder 对象中
bm25_encoder = BM25Encoder().load("bm25_values.json")
```

## 加载检索器

现在我们可以构建检索器了！

```python
retriever = PineconeHybridSearchRetriever(
    embeddings=embeddings, sparse_encoder=bm25_encoder, index=index
)
```

## 添加文本（如果需要）

我们可以选择将文本添加到检索器中（如果它们还没有在其中）

```python
retriever.add_texts(["foo", "bar", "world", "hello"])
```

```output
100%|██████████| 1/1 [00:02<00:00,  2.27s/it]
```

## 使用检索器

现在我们可以使用检索器了！

```python
result = retriever.invoke("foo")
```

```python
result[0]
```

```output
Document(page_content='foo', metadata={})
```