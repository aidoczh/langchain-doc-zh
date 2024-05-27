# OpenSearch

> [OpenSearch](https://opensearch.org/) 是一个可扩展、灵活、可扩展的开源软件套件，用于搜索、分析和可观察性应用，根据 Apache 2.0 许可证授权。`OpenSearch` 是基于 `Apache Lucene` 的分布式搜索和分析引擎。

在这个笔记本中，我们将演示使用 `OpenSearch` 向量存储的 `SelfQueryRetriever`。

## 创建 OpenSearch 向量存储

首先，我们需要创建一个 `OpenSearch` 向量存储，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** self-query retriever 需要你已安装 `lark` (`pip install lark`)。我们还需要 `opensearch-py` 包。

```python
%pip install --upgrade --quiet  lark opensearch-py
```

```python
import getpass
import os
from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
embeddings = OpenAIEmbeddings()
```

```output
OpenAI API Key: ········
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一片混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法",
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
            "rating": 9.9,
            "director": "安德烈·塔可夫斯基",
            "genre": "科幻",
        },
    ),
]
vectorstore = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    index_name="opensearch-self-query-demo",
    opensearch_url="http://localhost:9200",
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

## 测试一下

现在我们可以尝试使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的一些电影")
```

```output
query='dinosaur' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活了恐龙，然后一片混乱', metadata={'year': 1993, 'rating': 7.7, 'genre': '科幻'}),
 Document(page_content='玩具活了起来，并且玩得很开心', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中...', metadata={'year': 2010, 'director': '克里斯托弗·诺兰', 'rating': 8.2}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔可夫斯基', 'genre': '科幻'})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于 8.5 的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
```

```output
[文档(page_content='三名男子走进区域，三名男子走出区域', metadata={'年份': 1979, '评分': 9.9, '导演': '安德烈·塔可夫斯基', '类型': '科幻'}),
 文档(page_content='一位心理学家/侦探在一系列梦境中迷失，其中包含梦中梦的概念，而《盗梦空间》重新利用了这个想法', metadata={'年份': 2006, '导演': '今敏', '评分': 8.6})]
```

```python
# 这个示例指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦尔格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```

```output
[Document(page_content='一群普通身材的女性非常健康，一些男人为她们倾心', metadata={'year': 2019, 'director': 'Greta Gerwig', 'rating': 8.3})]
```

```python
# 这个示例指定了一个复合过滤器
retriever.invoke("评分高于8.5的科幻电影有哪些")
```

```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='genre', value='science fiction')]) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': 'Andrei Tarkovsky', 'genre': 'science fiction'})]
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
# 这个示例只指定了一个相关查询
retriever.invoke("关于恐龙的两部电影是什么")
```

```output
query='dinosaur' filter=None limit=2
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'rating': 7.7, 'genre': 'science fiction'}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': 'animated'})]
```

## 复杂查询实例！

我们已经尝试了一些简单的查询，但更复杂的查询呢？让我们尝试一些更复杂的查询，利用 OpenSearch 的全部功能。

```python
retriever.invoke(
    "过去30年中有哪些关于动画玩具的动画片或喜剧电影发布了"
)
```

```output
query='animated toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Operation(operator=<Operator.OR: 'or'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated'), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='comedy')]), Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='year', value=1990)]) limit=None
```

```output
[Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': 'animated'})]
```

```python
vectorstore.client.indices.delete(index="opensearch-self-query-demo")
```

```output
{'acknowledged': True}
```