# MyScale

[MyScale](https://docs.myscale.com/en/) 是一个集成的向量数据库。您可以通过 SQL 和 LangChain 访问您的数据库。

`MyScale` 可以利用[各种数据类型和过滤函数](https://blog.myscale.com/2023/06/06/why-integrated-database-solution-can-boost-your-llm-apps/#filter-on-anything-without-constraints)。无论您是扩展数据还是将系统扩展到更广泛的应用程序，它都将提升您的 LLM 应用。

在笔记本中，我们将演示 `SelfQueryRetriever` 包装在一个 `MyScale` 向量存储器周围，并附加了一些我们为 LangChain 贡献的额外部分。

简而言之，它可以归纳为 4 点：

1. 添加 `contain` 比较器以匹配任何列表，如果有多个元素匹配

2. 添加 `timestamp` 数据类型以进行日期时间匹配（ISO 格式或 YYYY-MM-DD）

3. 添加 `like` 比较器以进行字符串模式搜索

4. 添加任意函数功能

## 创建 MyScale 向量存储器

MyScale 已经集成到 LangChain 中一段时间了。因此，您可以按照[此笔记本](/docs/integrations/vectorstores/myscale)创建自己的向量存储器以进行自查询检索。

**注意：** 所有自查询检索器都需要您安装 `lark` (`pip install lark`)。我们使用 `lark` 进行语法定义。在您继续下一步之前，我们还想提醒您需要安装 `clickhouse-connect` 以与您的 MyScale 后端交互。

```python
%pip install --upgrade --quiet  lark clickhouse-connect
```

在本教程中，我们遵循其他示例的设置，并使用 `OpenAIEmbeddings`。请记得获取 OpenAI API 密钥以有效访问 LLM。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
os.environ["MYSCALE_HOST"] = getpass.getpass("MyScale URL:")
os.environ["MYSCALE_PORT"] = getpass.getpass("MyScale 端口:")
os.environ["MYSCALE_USERNAME"] = getpass.getpass("MyScale 用户名:")
os.environ["MYSCALE_PASSWORD"] = getpass.getpass("MyScale 密码:")
```

```python
from langchain_community.vectorstores import MyScale
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

## 创建一些示例数据

正如您所看到的，我们创建的数据与其他自查询检索器有一些不同。我们用 `date` 替换了关键字 `year`，这样您就可以更精细地控制时间戳。我们还将关键字 `gerne` 的类型更改为字符串列表，LLM 可以使用新的 `contain` 比较器来构建过滤器。我们还为过滤器提供了 `like` 比较器和任意函数支持，这将在接下来的几个单元格中介绍。

现在让我们先看一下数据。

```python
docs = [
    Document(
        page_content="一群科学家带回恐龙，混乱不堪",
        metadata={"date": "1993-07-02", "rating": 7.7, "genre": ["science fiction"]},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中...",
        metadata={"date": "2010-12-30", "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法",
        metadata={"date": "2006-04-23", "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通大小的女性非常健康，一些男性对她们倾慕",
        metadata={"date": "2019-08-22", "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具活了过来，并且玩得很开心",
        metadata={"date": "1995-02-11", "genre": ["animated"]},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={
            "date": "1979-09-10",
            "director": "安德烈·塔可夫斯基",
            "genre": ["science fiction", "adventure"],
            "rating": 9.9,
        },
    ),
]
vectorstore = MyScale.from_documents(
    docs,
    embeddings,
)
```

## 创建我们的自查询检索器

就像其他检索器一样... 简单而友好。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI
metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型",
        type="list[string]",
    ),
    # 如果你想包含列表的长度，只需将其定义为一个新的列
    # 这将教会 LLM 在构建过滤器时使用它作为列。
    AttributeInfo(
        name="length(genre)",
        description="电影类型的长度",
        type="integer",
    ),
    # 现在你可以将列定义为时间戳。只需将类型设置为 timestamp。
    AttributeInfo(
        name="date",
        description="电影发布的日期",
        type="timestamp",
    ),
    AttributeInfo(
        name="director",
        description="电影导演的姓名",
        type="string",
    ),
    AttributeInfo(
        name="rating", description="电影的评分（1-10）", type="float"
    ),
]
document_content_description = "电影的简要概述"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 使用自查询检索器的现有功能进行测试

现在我们可以尝试使用我们的检索器了！

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有关恐龙的电影有哪些")
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("Greta Gerwig导演过关于女性的电影吗")
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分高于8.5的科幻电影？")
```

```python
# 这个例子指定了一个查询和一个复合过滤器
retriever.invoke(
    "1990年后但2005年前的关于玩具的电影，最好是动画片"
)
```

# 等一下... 还有什么其他功能吗？

使用MyScale的自查询检索器还可以做更多的事情！让我们来看看。

```python
# 您可以使用length(genres)来做任何您想做的事情
retriever.invoke("有多于1个类型的电影有哪些？")
```

```python
# 精确到日期和时间？您已经有了。
retriever.invoke("1995年2月后发布的电影有哪些？")
```

```python
# 不知道您的确切过滤器应该是什么？使用字符串模式匹配！
retriever.invoke("名字像Andrei的电影有哪些？")
```

```python
# 包含适用于列表：因此您可以使用包含比较器来匹配列表！
retriever.invoke("同时具有科幻和冒险类型的电影有哪些？")
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
    enable_limit=True,
    verbose=True,
)
```

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有关恐龙的两部电影")
```