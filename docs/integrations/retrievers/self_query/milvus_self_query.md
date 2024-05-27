# Milvus

[Milvus](https://milvus.io/docs/overview.md) 是一个数据库，用于存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大规模嵌入向量。

在本教程中，我们将演示使用 `Milvus` 向量存储的 `SelfQueryRetriever`。

## 创建 Milvus 向量存储

首先，我们需要创建一个 Milvus 向量存储，并将一些数据填充到其中。我们创建了一个包含电影摘要的小型演示文档集。

我使用的是 Milvus 的云版本，因此我需要提供 `uri` 和 `token`。

注意：自查询检索器需要您安装 `lark` (`pip install lark`)。我们还需要 `pymilvus` 包。

```python
%pip install --upgrade --quiet  lark
```

```python
%pip install --upgrade --quiet  pymilvus
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import os
OPENAI_API_KEY = "使用您的 OpenAI 密钥:)"
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain_community.vectorstores import Milvus
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活恐龙，混乱不断",
        metadata={"year": 1993, "rating": 7.7, "genre": "动作"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的梦中...",
        metadata={"year": 2010, "genre": "惊悚", "rating": 8.2},
    ),
    Document(
        page_content="一群普通身材的女性极具魅力，一些男性为她们倾心",
        metadata={"year": 2019, "rating": 8.3, "genre": "剧情"},
    ),
    Document(
        page_content="三个人走进地带，三个人走出地带",
        metadata={"year": 1979, "rating": 9.9, "genre": "科幻"},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重复了这个想法",
        metadata={"year": 2006, "genre": "惊悚", "rating": 9.0},
    ),
    Document(
        page_content="玩具活了起来，并且玩得不亦乐乎",
        metadata={"year": 1995, "genre": "动画", "rating": 9.3},
    ),
]
vector_store = Milvus.from_documents(
    docs,
    embedding=embeddings,
    connection_args={"uri": "使用您的 uri:)", "token": "使用您的 token:)"},
)
```

## 创建我们的自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供有关我们的文档支持的元数据字段的一些信息以及文档内容的简短描述。

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
        description="电影发行年份",
        type="整数",
    ),
    AttributeInfo(
        name="rating", description="电影的 1-10 评分", type="浮点数"
    ),
]
document_content_description = "电影简要摘要"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vector_store, document_content_description, metadata_field_info, verbose=True
)
```

## 测试

现在我们可以尝试实际使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的电影有哪些")
```

```output
query='dinosaur' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'rating': 7.7, 'genre': '动作'}),
 Document(page_content='玩具活了起来，并且玩得不亦乐乎', metadata={'year': 1995, 'rating': 9.3, 'genre': '动画'}),
 Document(page_content='三个人走进地带，三个人走出地带', metadata={'year': 1979, 'rating': 9.9, 'genre': '科幻'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重复了这个想法', metadata={'year': 2006, 'rating': 9.0, 'genre': '惊悚'})]
```

```python
# 这个例子指定了一个过滤器
retriever.invoke("有哪些评分高于 9 的电影？")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=9) limit=None
```

```output
[Document(page_content='玩具活了起来，并且玩得不亦乐乎', metadata={'year': 1995, 'rating': 9.3, 'genre': '动画'}),
 Document(page_content='三个人走进地带，三个人走出地带', metadata={'year': 1979, 'rating': 9.9, 'genre': '科幻'})]
```

```python
# 这个例子只指定了一个查询和一个过滤条件
retriever.invoke("我想看一部关于玩具的电影，评分高于9分")
```

```output
查询='玩具' 过滤=比较(比较器=<Comparator.GT: 'gt'>, 属性='评分', 值=9) 限制=None
```

```output
[Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'年份': 1995, '评分': 9.3, '类型': '动画'}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'年份': 1979, '评分': 9.9, '类型': '科幻'})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分很高（大于或等于9）的惊悚电影？")
```

```output
查询=' ' 过滤=操作(操作符=<Operator.AND: 'and'>, 参数=[比较(比较器=<Comparator.EQ: 'eq'>, 属性='类型', 值='惊悚'), 比较(比较器=<Comparator.GTE: 'gte'>, 属性='评分', 值=9)]) 限制=None
```

```output
[Document(page_content='一位心理学家/侦探在一系列梦境中迷失，电影《盗梦空间》重新利用了这个概念', metadata={'年份': 2006, '评分': 9.0, '类型': '惊悚'})]
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前关于恐龙的电影，最好有很多动作"
)
```

```output
查询='恐龙' 过滤=操作(操作符=<Operator.AND: 'and'>, 参数=[比较(比较器=<Comparator.GT: 'gt'>, 属性='年份', 值=1990), 比较(比较器=<Comparator.LT: 'lt'>, 属性='年份', 值=2005), 比较(比较器=<Comparator.EQ: 'eq'>, 属性='类型', 值='动作')]) 限制=None
```

```output
[Document(page_content='一群科学家复活了恐龙，混乱开始蔓延', metadata={'年份': 1993, '评分': 7.7, '类型': '动作'})]
```

## 过滤器 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vector_store,
    document_content_description,
    metadata_field_info,
    verbose=True,
    enable_limit=True,
)
```

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有关恐龙的两部电影是什么？")
```

```output
查询='恐龙' 过滤=None 限制=2
```

```output
[Document(page_content='一群科学家复活了恐龙，混乱开始蔓延', metadata={'年份': 1993, '评分': 7.7, '类型': '动作'}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'年份': 1995, '评分': 9.3, '类型': '动画'})]
```