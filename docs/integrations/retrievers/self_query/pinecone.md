# Pinecone

[Pinecone](https://docs.pinecone.io/docs/overview) 是一个功能广泛的向量数据库。

在本教程中，我们将使用 `SelfQueryRetriever` 和 `Pinecone` 向量存储演示。

## 创建 Pinecone 索引

首先，我们需要创建一个 `Pinecone` 向量存储，并将一些数据添加到其中。我们创建了一个包含电影摘要的小型演示文档集。

要使用 Pinecone，您必须安装 `pinecone` 包，并且必须拥有 API 密钥和环境。这里是[安装说明](https://docs.pinecone.io/docs/quickstart)。

**注意：** 自查询检索器需要您安装 `lark` 包。

```python
%pip install --upgrade --quiet  lark
```

```python
%pip install --upgrade --quiet pinecone-notebooks pinecone-client==3.2.2
```

```python
# 连接到 Pinecone 并获取 API 密钥。
from pinecone_notebooks.colab import Authenticate
Authenticate()
import os
api_key = os.environ["PINECONE_API_KEY"]
```

```output
/Users/harrisonchase/.pyenv/versions/3.9.1/envs/langchain/lib/python3.9/site-packages/pinecone/index.py:4: TqdmExperimentalWarning: Using `tqdm.autonotebook.tqdm` in notebook mode. Use `tqdm.tqdm` instead to force console mode (e.g. in jupyter console)
  from tqdm.autonotebook import tqdm
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
```

```python
from pinecone import Pinecone, ServerlessSpec
api_key = os.getenv("PINECONE_API_KEY") or "PINECONE_API_KEY"
index_name = "langchain-self-retriever-demo"
pc = Pinecone(api_key=api_key)
```

```python
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore
embeddings = OpenAIEmbeddings()
# 创建新索引
if index_name not in pc.list_indexes().names():
    pc.create_index(
        name=index_name,
        dimension=1536,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
    )
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后造成了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": ["动作", "科幻"]},
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
        page_content="一群普通身材的女性非常纯真，一些男性对她们念念不忘",
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
            "genre": ["科幻", "惊悚"],
            "rating": 9.9,
        },
    ),
]
vectorstore = PineconeVectorStore.from_documents(
    docs, embeddings, index_name="langchain-self-retriever-demo"
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器了。为此，我们需要提供一些关于文档支持的元数据字段的信息以及文档内容的简短描述。

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
        name="rating", description="电影的评分（1-10）", type="浮点数"
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
retriever.invoke("有关恐龙的电影有哪些")
```

```output
query='dinosaur' filter=None
```

```output
[文档(page_content='一群科学家复活恐龙，混乱不断', metadata={'类型': ['动作', '科幻'], '评分': 7.7, '年份': 1993.0}),
 文档(page_content='玩具活了起来，玩得不亦乐乎', metadata={'类型': '动画', '年份': 1995.0}),
 文档(page_content='一位心理学家/侦探在一系列梦境中迷失，其中梦中梦的概念被《盗梦空间》重新利用', metadata={'导演': '今敏', '评分': 8.6, '年份': 2006.0}),
 文档(page_content='莱昂纳多在一个梦中的梦中的梦中迷失，而这个概念被《盗梦空间》再次利用...', metadata={'导演': '克里斯托弗·诺兰', '评分': 8.2, '年份': 2010.0})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)
```

```output
[Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'director': 'Satoshi Kon', 'rating': 8.6, 'year': 2006.0}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'director': 'Andrei Tarkovsky', 'genre': ['science fiction', 'thriller'], 'rating': 9.9, 'year': 1979.0})]
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig')
```

```output
[Document(page_content='一群普通身材的女性非常健康，一些男人向往她们', metadata={'director': 'Greta Gerwig', 'rating': 8.3, 'year': 2019.0})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分很高（超过8.5分）的科幻电影？")
```

```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='science fiction'), Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)])
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'director': 'Andrei Tarkovsky', 'genre': ['science fiction', 'thriller'], 'rating': 9.9, 'year': 1979.0})]
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前的关于玩具的电影，最好是动画片"
)
```

```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990.0), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005.0), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated')])
```

```output
[Document(page_content='玩具活了起来，并且玩得不亦乐乎', metadata={'genre': 'animated', 'year': 1995.0})]
```

## Filter k

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
retriever.invoke("有关恐龙的两部电影是什么")
```