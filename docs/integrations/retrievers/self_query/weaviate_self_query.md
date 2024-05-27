# Weaviate

[Weaviate](https://weaviate.io/) 是一个开源的向量数据库。它允许您存储来自您喜爱的机器学习模型的数据对象和向量嵌入，并能够无缝地扩展到数十亿个数据对象。

在这个笔记本中，我们将演示包装在 Weaviate 向量存储周围的 SelfQueryRetriever。

## 创建 Weaviate 向量存储

首先，我们需要创建一个 Weaviate 向量存储，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：**自查询检索器需要您安装 `lark`（`pip install lark`）。我们还需要 `weaviate-client` 包。

```python
%pip install --upgrade --quiet  lark weaviate-client
```

```python
from langchain_community.vectorstores import Weaviate
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后造成了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦中的梦中的梦中，而《盗梦空间》重新使用了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通身材的女性非常纯真，一些男性对她们倾心",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具们活了起来，并且玩得很开心",
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
vectorstore = Weaviate.from_documents(
    docs, embeddings, weaviate_url="http://127.0.0.1:8080"
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器了。为此，我们需要提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简短描述。

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
        description="电影的发行年份",
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

现在我们可以尝试使用我们的检索器了！

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有关恐龙的一些电影")
```

```output
query='dinosaur' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活了恐龙，然后造成了混乱', metadata={'genre': '科幻', 'rating': 7.7, 'year': 1993}),
 Document(page_content='玩具们活了起来，并且玩得很开心', metadata={'genre': '动画', 'rating': None, 'year': 1995}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'genre': '科幻', 'rating': 9.9, 'year': 1979}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦中的梦中的梦中，而《盗梦空间》重新使用了这个想法', metadata={'genre': None, 'rating': 8.6, 'year': 2006})]
```

```python
# 这个例子指定了查询和过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```

```output
[Document(page_content='一群普通身材的女性非常纯真，一些男性对她们倾心', metadata={'genre': None, 'rating': 8.3, 'year': 2019})]
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

上述代码创建了一个 `SelfQueryRetriever` 对象，该对象使用了一个名为 `llm` 的模型，一个名为 `vectorstore` 的向量存储，一个名为 `document_content_description` 的文档内容描述，以及一个名为 `metadata_field_info` 的元数据字段信息。此外，还启用了限制功能，并打开了详细输出模式。

```python
# 这个例子只指定了一个相关查询
retriever.invoke("关于恐龙的两部电影是什么")
```

```output
query='dinosaur' filter=None limit=2
```

```output
[Document(page_content='一群科学家复活恐龙，结果引发了混乱', metadata={'genre': '科幻', 'rating': 7.7, 'year': 1993}),
 Document(page_content='玩具活了起来，并且玩得不亦乐乎', metadata={'genre': '动画', 'rating': None, 'year': 1995})]
```