# Astra DB (Cassandra)

[DataStax Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 是一个基于 `Cassandra` 构建的无服务器向量数据库，通过易于使用的 JSON API 方便地提供。

在本教程中，我们将演示使用 `Astra DB` 向量存储的 `SelfQueryRetriever`。

## 创建 Astra DB 向量存储

首先，我们需要创建一个 Astra DB 向量存储，并用一些数据进行填充。我们创建了一个包含电影摘要的小型演示文档集。

注意：self-query retriever 需要你安装 `lark` (`pip install lark`)。我们还需要 `astrapy` 包。

```python
%pip install --upgrade --quiet lark astrapy langchain-openai
```

我们希望使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import os
from getpass import getpass
from langchain_openai.embeddings import OpenAIEmbeddings
os.environ["OPENAI_API_KEY"] = getpass("OpenAI API Key:")
embeddings = OpenAIEmbeddings()
```

创建 Astra DB 向量存储：

- API 端点看起来像 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`

- 令牌看起来像 `AstraCS:6gBhNmsk135....`

```python
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```

```python
from langchain_community.vectorstores import AstraDB
from langchain_core.documents import Document
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一片混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，一层梦套着一层梦...",
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
        page_content="玩具活了过来，并且玩得不亦乐乎",
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
vectorstore = AstraDB.from_documents(
    docs,
    embeddings,
    collection_name="astra_self_query_demo",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简短描述。

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
        name="rating", description="电影的1-10评分", type="浮点数"
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
retriever.invoke("有关恐龙的一些电影是什么？")
```

```python
# 这个例子指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```python
# 这个例子只指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗？")
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分很高（超过8.5）的科幻电影？")
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前关于玩具的电影，而且是动画的有哪些？"
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

以上代码创建了一个 `SelfQueryRetriever` 对象，并从 `llm` 中加载模型。`vectorstore` 是一个用于存储向量的对象，`document_content_description` 是描述文档内容的信息，`metadata_field_info` 是元数据字段的信息。`verbose=True` 表示打印详细信息，`enable_limit=True` 表示启用限制功能。

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("关于恐龙的两部电影是什么？")
```

## 清理

如果你想要完全从你的 Astra DB 实例中删除这个集合，运行以下命令。

_(你将会丢失其中存储的数据。)_

```python
vectorstore.delete_collection()
```