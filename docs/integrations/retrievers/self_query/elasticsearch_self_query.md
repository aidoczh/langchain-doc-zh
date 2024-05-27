# Elasticsearch

[Elasticsearch](https://www.elastic.co/elasticsearch/) 是一个分布式的、基于 REST 的搜索和分析引擎。它提供了一个分布式、多租户的全文搜索引擎，具有 HTTP web 接口和无模式的 JSON 文档。

在这个笔记本中，我们将演示使用 `Elasticsearch` 向量存储的 `SelfQueryRetriever`。

## 创建一个 Elasticsearch 向量存储

首先，我们需要创建一个 `Elasticsearch` 向量存储，并向其中添加一些数据。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** 自查询检索器需要你已安装 `lark`（`pip install lark`）。我们还需要 `elasticsearch` 包。

```python
%pip install --upgrade --quiet  U lark langchain langchain-elasticsearch
```

```output
WARNING: You are using pip version 22.0.4; however, version 23.3 is available.
You should consider upgrading via the '/Users/joe/projects/elastic/langchain/libs/langchain/.venv/bin/python3 -m pip install --upgrade pip' command.
```

```python
import getpass
import os
from langchain_core.documents import Document
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一团混乱发生了",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦境中的梦境中的梦境中...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通大小的女性非常纯真，一些男性对她们倾慕",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具活了起来，并且玩得不亦乐乎",
        metadata={"year": 1995, "genre": "动画"},
    ),
    Document(
        page_content="三个人走进这个地带，三个人走出这个地带",
        metadata={
            "year": 1979,
            "director": "安德烈·塔可夫斯基",
            "genre": "科幻",
            "rating": 9.9,
        },
    ),
]
vectorstore = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    index_name="elasticsearch-self-query-demo",
    es_url="http://localhost:9200",
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
        name="rating", description="电影的1-10评分", type="浮点数"
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
[Document(page_content='一群科学家复活了恐龙，然后一团混乱发生了', metadata={'year': 1993, 'rating': 7.7, 'genre': '科幻'}),
 Document(page_content='玩具活了起来，并且玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='三个人走进这个地带，三个人走出这个地带', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔可夫斯基', 'genre': '科幻'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.6})]
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
[文档(page_content='一群普通身材的女性非常健康，一些男性向往她们', metadata={'年份': 2019, '导演': '格蕾塔·葛韦格', '评分': 8.3})]
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
# 此示例仅指定了一个相关查询
retriever.invoke("关于恐龙的两部电影是什么")
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'rating': 7.7, 'genre': 'science fiction'}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': 'animated'})]
```

## 复杂查询实例！

我们已经尝试了一些简单的查询，但更复杂的查询呢？让我们尝试一些更复杂的查询，充分利用 Elasticsearch 的全部功能。

```python
retriever.invoke(
    "过去 30 年中有哪些关于动画玩具的动画片或喜剧电影？"
)
```

```output
[Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': 'animated'})]
```

```python
vectorstore.client.indices.delete(index="elasticsearch-self-query-demo")
```