# Chroma

[Chroma](https://docs.trychroma.com/getting-started) 是一个用于构建带有嵌入向量的人工智能应用程序的向量数据库。

在笔记本中，我们将演示围绕着一个 `Chroma` 向量存储器的 `SelfQueryRetriever`。

## 创建 Chroma 向量存储器

首先，我们需要创建一个 Chroma 向量存储器，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** 自查询检索器要求您已安装 `lark` (`pip install lark`)。我们还需要 `langchain-chroma` 包。

```python
%pip install --upgrade --quiet  lark
```

```python
%pip install --upgrade --quiet  langchain-chroma
```

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```output
OpenAI API Key: ········
```

```python
from langchain_chroma import Chroma
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```

```python
docs = [
    Document(
        page_content="一群科学家复活恐龙，混乱不断",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在梦中的梦中的梦中的梦中...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个概念",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通身材的女性极具魅力，一些男性为她们倾心",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具活了起来，乐此不疲",
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
vectorstore = Chroma.from_documents(docs, embeddings)
```

```output
使用内嵌 DuckDB，无持久性：数据将是瞬时的
```

## 创建我们的自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供关于文档支持的元数据字段以及文档内容的简短描述。

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

现在我们可以尝试实际使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的一些电影")
```

```output
query='dinosaur' filter=None
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'rating': 7.7, 'genre': '科幻'}),
 Document(page_content='玩具活了起来，乐此不疲', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个概念', metadata={'year': 2006, 'director': '今敏', 'rating': 8.6}),
 Document(page_content='莱昂纳多·迪卡普里奥迷失在梦中的梦中的梦中的梦中...', metadata={'year': 2010, 'director': '克里斯托弗·诺兰', 'rating': 8.2})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)
```

```output
[文档(page_content='一位心理学家/侦探陷入了一连串的梦境中，其中包含了梦中的梦境，而《盗梦空间》重新使用了这个想法', metadata={'年份': 2006, '导演': 'Satoshi Kon', '评分': 8.6}),
 文档(page_content='三个人走进了禁区，三个人走出了禁区', metadata={'年份': 1979, '评分': 9.9, '导演': 'Andrei Tarkovsky', '类型': '科幻'})]
```

```python
# 这个示例指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦尔格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig')
```

```output
[Document(page_content='一群普通身材的女性非常健康，一些男性为她们倾倒', metadata={'year': 2019, 'director': 'Greta Gerwig', 'rating': 8.3})]
```

```python
# 这个示例指定了一个复合过滤器
retriever.invoke("评分高于8.5的科幻电影是什么")
```

```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='science fiction'), Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)])
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔可夫斯基', 'genre': 'science fiction'})]
```

```python
# 这个示例指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前关于玩具的电影，最好是动画片"
)
```

```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated')])
```

```output
[Document(page_content='玩具们活了起来，乐此不疲', metadata={'year': 1995, 'genre': 'animated'})]
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
query='dinosaur' filter=None
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'rating': 7.7, 'genre': 'science fiction'}),
 Document(page_content='玩具们活了起来，乐此不疲', metadata={'year': 1995, 'genre': 'animated'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，其中包含梦中梦的概念，而《盗梦空间》重新利用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.6}),
 Document(page_content='莱昂纳多迷失在一个梦中的梦中的梦中的世界中...', metadata={'year': 2010, 'director': '克里斯托弗·诺兰', 'rating': 8.2})]
```