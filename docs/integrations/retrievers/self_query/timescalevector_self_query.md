# Timescale Vector（Postgres）

>[Timescale Vector](https://www.timescale.com/ai) 是用于 AI 应用的 `PostgreSQL++`。它使您能够在 `PostgreSQL` 中高效地存储和查询数十亿个向量嵌入。

>

>[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 也被称为 `Postgres`，是一个免费开源的关系型数据库管理系统（RDBMS），强调可扩展性和 `SQL` 兼容性。

本文档展示了如何使用 Postgres 向量数据库（TimescaleVector）进行自查询。在本文档中，我们将演示包装在 TimescaleVector 向量存储周围的 SelfQueryRetriever。

## Timescale Vector 是什么？

**[Timescale Vector](https://www.timescale.com/ai) 是用于 AI 应用的 PostgreSQL++。**

Timescale Vector 使您能够在 `PostgreSQL` 中高效地存储和查询数百万个向量嵌入。

- 通过受 DiskANN 启发的索引算法，增强了 `pgvector` 在 10 亿个以上向量上的更快和更准确的相似性搜索。

- 通过自动基于时间的分区和索引，实现了快速的基于时间的向量搜索。

- 提供了一个熟悉的 SQL 接口，用于查询向量嵌入和关系数据。

Timescale Vector 是用于 AI 的云端 PostgreSQL，从 POC 到生产都能与您一起扩展：

- 通过使您能够在单个数据库中存储关系元数据、向量嵌入和时间序列数据，简化了操作。

- 基于坚实的 PostgreSQL 基础，具有企业级功能，如流式备份和复制、高可用性和行级安全性。

- 提供了企业级安全性和合规性，让您无忧无虑。

## 如何访问 Timescale Vector

Timescale Vector 可在 [Timescale](https://www.timescale.com/ai) 上的云端 PostgreSQL 平台上使用（目前没有自托管版本）。

LangChain 用户可以免费试用 Timescale Vector 90 天。

- 要开始使用，请[注册](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) Timescale，创建一个新的数据库，然后按照本文档的步骤进行操作！

- 有关更多详细信息和性能基准，请参阅[Timescale Vector 解释博客](https://www.timescale.com/blog/how-we-made-postgresql-the-best-vector-database/?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)。

- 有关在 Python 中使用 Timescale Vector 的更多详细信息，请参阅[安装说明](https://github.com/timescale/python-vector)。

## 创建 TimescaleVector 向量存储

首先，我们需要创建一个 Timescale Vector 向量存储，并使用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

注意：自查询检索器需要您安装 `lark`（`pip install lark`）。我们还需要 `timescale-vector` 包。

```python
%pip install --upgrade --quiet  lark
```

```python
%pip install --upgrade --quiet  timescale-vector
```

在此示例中，我们将使用 `OpenAIEmbeddings`，因此让我们加载您的 OpenAI API 密钥。

```python
# 通过读取本地的 .env 文件获取 OpenAI API 密钥
# .env 文件应包含以 `OPENAI_API_KEY=sk-` 开头的一行
import os
from dotenv import find_dotenv, load_dotenv
_ = load_dotenv(find_dotenv())
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
# 或者，使用 getpass 在提示符中输入密钥
# import os
# import getpass
# os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

要连接到您的 PostgreSQL 数据库，您需要服务 URI，可以在创建新数据库后下载的 cheatsheet 或 `.env` 文件中找到。

如果您还没有，请[注册 Timescale](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)，并创建一个新的数据库。

URI 的格式类似于：`postgres://tsdbadmin:<password>@<id>.tsdb.cloud.timescale.com:<port>/tsdb?sslmode=require`

```python
# 通过读取本地的 .env 文件获取服务 URL
# .env 文件应包含以 `TIMESCALE_SERVICE_URL=postgresql://` 开头的一行
_ = load_dotenv(find_dotenv())
TIMESCALE_SERVICE_URL = os.environ["TIMESCALE_SERVICE_URL"]
# 或者，使用 getpass 在提示符中输入密钥
# import os
# import getpass
# TIMESCALE_SERVICE_URL = getpass.getpass("Timescale Service URL:")
```

```python
from langchain_community.vectorstores.timescalevector import TimescaleVector
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

以下是我们将在此演示中使用的示例文档。数据是关于电影的，包含有关特定电影的内容和元数据字段。

```python
docs = [
    Document(
        page_content="一群科学家复活恐龙，引发了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的梦中...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦中的梦中的梦中，而《盗梦空间》重复了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通大小的女性非常纯真，一些男性对她们倾心",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具们活了起来，并且玩得不亦乐乎",
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
```

最后，我们将创建我们的时间序列向量 vectorstore。请注意，集合名称将是存储文档的 PostgreSQL 表的名称。

```python
COLLECTION_NAME = "langchain_self_query_demo"
vectorstore = TimescaleVector.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    service_url=TIMESCALE_SERVICE_URL,
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于文档支持的元数据字段的信息以及文档内容的简要描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI
# 向 LLM 提供有关元数据字段的信息
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
        name="rating", description="电影的评分（1-10）", type="浮点数"
    ),
]
document_content_description = "电影的简要概述"
# 从 LLM 实例化自查询检索器
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 使用 Timescale Vector 进行自查询检索

现在我们可以尝试实际使用我们的检索器！

运行下面的查询，并注意您可以用自然语言指定查询、过滤器、复合过滤器（带有 AND、OR 的过滤器），自查询检索器将把该查询转换为 SQL 并在 Timescale Vector（Postgres）向量存储中执行搜索。

这展示了自查询检索器的强大功能。您可以使用它在向量存储上执行复杂搜索，而无需您或您的用户直接编写任何 SQL！

```python
# 此示例仅指定相关查询
retriever.invoke("有关恐龙的一些电影")
```

```output
/Users/avtharsewrathan/sideprojects2023/timescaleai/tsv-langchain/langchain/libs/langchain/langchain/chains/llm.py:275: UserWarning: The predict_and_parse method is deprecated, instead pass an output parser directly to LLMChain.
  warnings.warn(
``````output
query='dinosaur' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'genre': '科幻', 'rating': 7.7}),
 Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'genre': '科幻', 'rating': 7.7}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'})]
```

```python
# 此示例仅指定过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': '科幻', 'rating': 9.9, 'director': '安德烈·塔尔科夫斯基'}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': '科幻', 'rating': 9.9, 'director': '安德烈·塔尔科夫斯基'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，梦中梦中梦，而《盗梦空间》重新利用了这个想法', metadata={'year': 2006, 'rating': 8.6, 'director': '今敏'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，梦中梦中梦，而《盗梦空间》重新利用了这个想法', metadata={'year': 2006, 'rating': 8.6, 'director': '今敏'})]
```

```python
# 此示例指定查询和过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```

```output
[Document(page_content='一群普通身材的女性非常纯真，一些男性向她们倾慕', metadata={'year': 2019, 'rating': 8.3, 'director': '格蕾塔·葛韦格'}),
 Document(page_content='一群普通身材的女性非常纯真，一些男性向她们倾慕', metadata={'year': 2019, 'rating': 8.3, 'director': '格蕾塔·葛韦格'})]
```

```python
# 此示例指定复合过滤器
```

```markdown
retriever.invoke("评分高于8.5的科幻电影有哪些？")
```

```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='science fiction')]) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': 'science fiction', 'rating': 9.9, 'director': 'Andrei Tarkovsky'}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': 'science fiction', 'rating': 9.9, 'director': 'Andrei Tarkovsky'})]
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年之后但2005年之前的关于玩具的电影，最好是动画片"
)
```

```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated')]) limit=None
```

```output
[Document(page_content='玩具们活了过来，并且玩得不亦乐乎', metadata={'year': 1995, 'genre': 'animated'})]
```

### 过滤器 k

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
# 这个例子指定了一个带有 LIMIT 值的查询
retriever.invoke("有关恐龙的两部电影是什么")
```

```output
query='dinosaur' filter=None limit=2
```

```output

[Document(page_content='一群科学家复活了恐龙，然后一系列麻烦接踵而至', metadata={'year': 1993, 'genre': 'science fiction', 'rating': 7.7}),

 Document(page_content='一群科学家复活了恐龙，然后一系列麻烦接踵而至', metadata={'year': 1993, 'genre': 'science fiction', 'rating': 7.7})]