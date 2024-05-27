# 深湖

>[深湖](https://www.activeloop.ai) 是一个用于构建人工智能应用程序的多模态数据库。

>[Deep Lake](https://github.com/activeloopai/deeplake) 是一个用于人工智能的数据库。

>可以存储向量、图像、文本、视频等数据。可与LLMs/LangChain一起使用。可以存储、查询、版本控制和可视化任何人工智能数据。可以实时将数据流式传输到PyTorch/TensorFlow。

在笔记本中，我们将演示包装在`Deep Lake`向量存储器周围的`SelfQueryRetriever`。

## 创建一个Deep Lake向量存储器

首先，我们需要创建一个Deep Lake向量存储器，并使用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：**自查询检索器需要您安装`lark`（`pip install lark`）。我们还需要`deeplake`包。

```python
%pip install --upgrade --quiet  lark
```
```python
# 如果某些查询失败，请考虑手动安装libdeeplake
%pip install --upgrade --quiet  libdeeplake
```

我们想使用`OpenAIEmbeddings`，所以我们需要获取OpenAI API密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
os.environ["ACTIVELOOP_TOKEN"] = getpass.getpass("Activeloop token:")
```
```python
from langchain_community.vectorstores import DeepLake
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```
```python
docs = [
    Document(
        page_content="一群科学家带回了恐龙，然后造成了混乱",
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
        page_content="一群普通大小的女人非常纯真，一些男人对她们念念不忘",
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
username_or_org = "<USERNAME_OR_ORG>"
vectorstore = DeepLake.from_documents(
    docs,
    embeddings,
    dataset_path=f"hub://{username_or_org}/self_queery",
    overwrite=True,
)
```
```output
您的Deep Lake数据集已成功创建！
``````output
/
``````output
Dataset(path='hub://adilkhan/self_queery', tensors=['embedding', 'id', 'metadata', 'text'])
  tensor      htype      shape     dtype  compression
  -------    -------    -------   -------  ------- 
 embedding  embedding  (6, 1536)  float32   None   
    id        text      (6, 1)      str     None   
 metadata     json      (6, 1)      str     None   
   text       text      (6, 1)      str     None
``````output
```

## 创建我们的自查询检索器

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
/home/ubuntu/langchain_activeloop/langchain/libs/langchain/langchain/chains/llm.py:279: UserWarning: The predict_and_parse method is deprecated, instead pass an output parser directly to LLMChain.
  warnings.warn(
````

查询='恐龙' 过滤=None 限制=None

````
```output

[Document(page_content='一群科学家复活了恐龙，引发了混乱', metadata={'year': 1993, 'rating': 7.7, 'genre': '科幻'}),

 Document(page_content='玩具们活了起来，并且玩得很开心', metadata={'year': 1995, 'genre': '动画'}),

 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔尔科夫斯基', 'genre': '科幻'}),

 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而电影《盗梦空间》则重新利用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.6})]

```
```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```
```output

query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None

```
```output

[Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而电影《盗梦空间》则重新利用了这个想法', metadata={'year': 2006, 'director': '今敏', 'rating': 8.6}),

 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔尔科夫斯基', 'genre': '科幻'})]

```
```python
# 这个例子指定了查询和过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```
```output

query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None

```
```output

[Document(page_content='一群普通身材的女性非常纯真，一些男性对她们倾心', metadata={'year': 2019, 'director': '格蕾塔·葛韦格', 'rating': 8.3})]

```
```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有没有一部评分高于8.5的科幻电影？")
```
```output

query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='科幻')]) limit=None

```
```output

[Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979, 'rating': 9.9, 'director': '安德烈·塔尔科夫斯基', 'genre': '科幻'})]

```
```python
# 这个例子指定了查询和复合过滤器
retriever.invoke(
    "1990年后但在2005年前的关于玩具的电影，最好是动画片"
)
```
```output

query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='动画')]) limit=None

```
```output

[Document(page_content='玩具们活了起来，并且玩得很开心', metadata={'year': 1995, 'genre': '动画'})]

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
retriever.invoke("有关恐龙的两部电影")
```
```output

query='dinosaur' filter=None limit=2

```
```output

[Document(page_content='一群科学家复活了恐龙，引发了混乱', metadata={'year': 1993, 'rating': 7.7, 'genre': '科幻'}),

 Document(page_content='玩具们活了起来，并且玩得很开心', metadata={'year': 1995, 'genre': '动画'})]

```