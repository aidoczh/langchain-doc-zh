# PGVector（Postgres）

[PGVector](https://github.com/pgvector/pgvector) 是一个用于 `Postgres` 数据库的向量相似性搜索包。

在这个笔记本中，我们将演示围绕 `PGVector` 向量存储器的 `SelfQueryRetriever`。

## 创建 PGVector 向量存储器

首先，我们需要创建一个 PGVector 向量存储器，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** self-query retriever 需要你已安装 `lark`（`pip install lark`）。我们还需要 `` 包。

```python
%pip install --upgrade --quiet  lark pgvector psycopg2-binary
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.vectorstores import PGVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
collection = "Name of your collection"
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一系列混乱事件发生了",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，梦中的梦中的梦中的...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重复了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通身材的女性非常健康，一些男性对她们倾慕",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具活了起来，并且玩得很开心",
        metadata={"year": 1995, "genre": "动画"},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={
            "year": 1979,
            "director": "安德烈·塔可夫斯基",
            "genre": "科幻",
            "rating": 9.9,
        },
    ),
]
vectorstore = PGVector.from_documents(
    docs,
    embeddings,
    collection_name=collection,
)
```

## 创建我们的自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI
metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型",
        type="字符串或字符串列表",
    ),
    AttributeInfo(
        name="year",
        description="电影上映的年份",
        type="整数",
    ),
    AttributeInfo(
        name="director",
        description="电影导演的姓名",
        type="字符串",
    ),
    AttributeInfo(
        name="rating", description="电影的 1-10 评分", type="浮点数"
    ),
]
document_content_description = "电影的简要摘要"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试

现在我们可以尝试使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的一些电影")
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于 8.5 的电影")
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分高于 8.5 的科幻电影？")
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990 年后但在 2005 年前的关于玩具的电影，最好是动画片"
```

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的两部电影")
```