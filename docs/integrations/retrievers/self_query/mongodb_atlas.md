# MongoDB Atlas

>[MongoDB Atlas](https://www.mongodb.com/) 是一种可以用作向量数据库的文档数据库。

在本教程中，我们将使用 `SelfQueryRetriever` 和 `MongoDB Atlas` 向量存储进行演示。

## 创建 MongoDB Atlas 向量存储

首先，我们需要创建一个 MongoDB Atlas 向量存储，并用一些数据填充它。我们已经创建了一组包含电影摘要的小型演示文档。

注意：自查询检索器需要您安装 `lark`（`pip install lark`）。我们还需要 `pymongo` 包。

```python
%pip install --upgrade --quiet  lark pymongo
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import os
OPENAI_API_KEY = "使用您的 OpenAI 密钥"
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from pymongo import MongoClient
CONNECTION_STRING = "使用您的 MongoDB Atlas 连接字符串"
DB_NAME = "您的 MongoDB Atlas 数据库名称"
COLLECTION_NAME = "数据库中的集合名称"
INDEX_NAME = "在集合上定义的搜索索引名称"
MongoClient = MongoClient(CONNECTION_STRING)
collection = MongoClient[DB_NAME][COLLECTION_NAME]
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后造成了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "动作"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的梦中...",
        metadata={"year": 2010, "genre": "惊悚", "rating": 8.2},
    ),
    Document(
        page_content="一群普通身材的女人非常纯真，一些男人对她们念念不忘",
        metadata={"year": 2019, "rating": 8.3, "genre": "剧情"},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={"year": 1979, "rating": 9.9, "genre": "科幻"},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦中的梦中的梦中，而《盗梦空间》重新使用了这个概念",
        metadata={"year": 2006, "genre": "惊悚", "rating": 9.0},
    ),
    Document(
        page_content="玩具们活了起来，并且玩得很开心",
        metadata={"year": 1995, "genre": "动画", "rating": 9.3},
    ),
]
vectorstore = MongoDBAtlasVectorSearch.from_documents(
    docs,
    embeddings,
    collection=collection,
    index_name=INDEX_NAME,
)
```

现在，让我们在您的集群上创建一个向量搜索索引。在下面的示例中，`embedding` 是包含嵌入向量的字段的名称。请参考[文档](https://www.mongodb.com/docs/atlas/atlas-search/field-types/knn-vector)以获取有关如何定义 Atlas 向量搜索索引的更多详细信息。

您可以将索引命名为 `{COLLECTION_NAME}`，并在命名空间 `{DB_NAME}.{COLLECTION_NAME}` 上创建索引。最后，在 MongoDB Atlas 的 JSON 编辑器中写入以下定义：

```json
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "embedding": {
        "dimensions": 1536,
        "similarity": "cosine",
        "type": "knnVector"
      },
      "genre": {
        "type": "token"
      },
      "ratings": {
        "type": "number"
      },
      "year": {
        "type": "number"
      }
    }
  }
}
```

## 创建自查询检索器

现在，我们可以实例化我们的检索器。为此，我们需要提前提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI
metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型",
        type="字符串",
    ),
    AttributeInfo(
        name="year",
        description="电影的发行年份",
        type="整数",
    ),
    AttributeInfo(
        name="rating", description="电影的评分（1-10）", type="浮点数"
    ),
]
document_content_description = "电影的简要摘要"
```

```python
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试一下

现在，我们可以尝试使用我们的检索器了！

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有关恐龙的一些电影")
```

```python
# 这个例子指定了一个过滤器
retriever.invoke("一些评分很高的电影（超过9分）")
```

```python
# 这个示例只指定了一个查询和一个过滤器
retriever.invoke("我想看一部关于玩具的电影，评分高于9分")
```

```python
# 这个示例指定了一个复合过滤器
retriever.invoke("高评分（大于或等于9分）的惊悚电影是什么？")
```

```python
# 这个示例指定了一个查询和复合过滤器
retriever.invoke(
    "1990年之后但2005年之前关于恐龙的电影，最好有很多动作"
)
```

## 过滤器 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    verbose=True,
    enable_limit=True,
)
```

```python
# 这个示例只指定了一个相关查询
retriever.invoke("有关恐龙的两部电影是什么？")
```