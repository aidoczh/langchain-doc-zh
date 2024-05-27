# DashVector

[DashVector](https://help.aliyun.com/document_detail/2510225.html) 是一种完全托管的向量数据库服务，支持高维稠密和稀疏向量、实时插入和过滤搜索。它被设计为能够自动扩展，并且可以适应不同的应用需求。向量检索服务 `DashVector` 基于由 `DAMO Academy` 独立开发的高效向量引擎 `Proxima` 核心，并提供了具有水平扩展能力的云原生、完全托管的向量检索服务。`DashVector` 通过简单易用的 SDK/API 接口暴露其强大的向量管理、向量查询和其他多样化的能力，可以被上层 AI 应用快速集成，从而提供包括大型模型生态、多模态 AI 搜索、分子结构分析等多种应用场景所需的高效向量检索能力。

在这个笔记本中，我们将演示使用 `DashVector` 向量存储的 `SelfQueryRetriever`。

## 创建 DashVector 向量存储

首先，我们需要创建一个 `DashVector` 向量存储，并向其中添加一些数据。我们创建了一个包含电影摘要的小型演示文档集。

要使用 DashVector，您必须安装 `dashvector` 包，并且必须拥有 API 密钥和环境。这里是[安装说明](https://help.aliyun.com/document_detail/2510223.html)。

注意：自查询检索器要求您已安装 `lark` 包。

```python
%pip install --upgrade --quiet  lark dashvector
```

```python
import os
import dashvector
client = dashvector.Client(api_key=os.environ["DASHVECTOR_API_KEY"])
```

```python
from langchain_community.embeddings import DashScopeEmbeddings
from langchain_community.vectorstores import DashVector
from langchain_core.documents import Document
embeddings = DashScopeEmbeddings()
# 创建 DashVector 集合
client.create("langchain-self-retriever-demo", dimension=1536)
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一系列混乱发生了",
        metadata={"year": 1993, "rating": 7.7, "genre": "action"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，一层梦中的梦中的梦中的...",
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
        page_content="玩具活了过来，并且玩得很开心",
        metadata={"year": 1995, "genre": "animated"},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={
            "year": 1979,
            "director": "安德烈·塔可夫斯基",
            "genre": "science fiction",
            "rating": 9.9,
        },
    ),
]
vectorstore = DashVector.from_documents(
    docs, embeddings, collection_name="langchain-self-retriever-demo"
)
```

## 创建您的自查询检索器

现在，我们可以实例化我们的检索器。为此，我们需要提前提供有关我们的文档支持的元数据字段的一些信息，以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_community.llms import Tongyi
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
llm = Tongyi(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试

现在我们可以尝试实际使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的一些电影")
```

```output
query='dinosaurs' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活了恐龙，引发了混乱', metadata={'year': 1993, 'rating': 7.699999809265137, 'genre': '动作'}),
 Document(page_content='玩具活了起来，并且乐在其中', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的...', metadata={'year': 2010, 'director': '克里斯托弗·诺兰', 'rating': 8.199999809265137}),
 Document(page_content='一位心理学家/侦探迷失在一系列的梦中的梦中的梦中，而《盗梦空间》重新利用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.600000381469727})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'director': '安德烈·塔可夫斯基', 'rating': 9.899999618530273, 'genre': '科幻'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，其中有梦境，而《盗梦空间》重新使用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.600000381469727})]
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
query='格蕾塔·葛韦格' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='格蕾塔·葛韦格') limit=None
```

```output
[Document(page_content='一群普通身材的女性非常健康，一些男性对她们倾心', metadata={'year': 2019, 'director': '格蕾塔·葛韦格', 'rating': 8.300000190734863})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有没有一部评分很高（超过8.5）的科幻电影？")
```

```output
query='科幻' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='科幻'), Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)]) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'director': '安德烈·塔可夫斯基', 'rating': 9.899999618530273, 'genre': '科幻'})]
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
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的两部电影是什么")
```

```output
query='恐龙' filter=None limit=2
```

```output
[Document(page_content='一群科学家复活了恐龙，然后造成了混乱', metadata={'year': 1993, 'rating': 7.699999809265137, 'genre': '动作'}),
 Document(page_content='玩具们活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'})]
```