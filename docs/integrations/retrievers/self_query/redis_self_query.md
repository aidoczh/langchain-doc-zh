# Redis

>[Redis](https://redis.com) 是一个开源的键值存储系统，可以用作缓存、消息代理、数据库、向量数据库等等。

在本文中，我们将演示包装在 `Redis` 向量存储系统中的 `SelfQueryRetriever`。

## 创建 Redis 向量存储系统

首先，我们需要创建一个 Redis 向量存储系统，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** 自查询检索器需要您安装 `lark`（`pip install lark`）以及特定于集成的要求。

```python
%pip install --upgrade --quiet  redis redisvl langchain-openai tiktoken lark
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥：")
```
```python
from langchain_community.vectorstores import Redis
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```
```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后引发了混乱",
        metadata={
            "year": 1993,
            "rating": 7.7,
            "director": "史蒂文·斯皮尔伯格",
            "genre": "科幻",
        },
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的...",
        metadata={
            "year": 2010,
            "director": "克里斯托弗·诺兰",
            "genre": "科幻",
            "rating": 8.2,
        },
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列的梦中的梦中的梦中，而《盗梦空间》重新使用了这个概念",
        metadata={
            "year": 2006,
            "director": "今敏",
            "genre": "科幻",
            "rating": 8.6,
        },
    ),
    Document(
        page_content="一群普通大小的女人非常纯真，一些男人对她们念念不忘",
        metadata={
            "year": 2019,
            "director": "格蕾塔·葛韦格",
            "genre": "剧情",
            "rating": 8.3,
        },
    ),
    Document(
        page_content="玩具们活了起来，并且玩得很开心",
        metadata={
            "year": 1995,
            "director": "约翰·拉塞特",
            "genre": "动画",
            "rating": 9.1,
        },
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
```
```python
index_schema = {
    "tag": [{"name": "genre"}],
    "text": [{"name": "director"}],
    "numeric": [{"name": "year"}, {"name": "rating"}],
}
vectorstore = Redis.from_documents(
    docs,
    embeddings,
    redis_url="redis://localhost:6379",
    index_name="movie_reviews",
    index_schema=index_schema,
)
```
```output
`index_schema` does not match generated metadata schema.
If you meant to manually override the schema, please ignore this message.
index_schema: {'tag': [{'name': 'genre'}], 'text': [{'name': 'director'}], 'numeric': [{'name': 'year'}, {'name': 'rating'}]}
generated_schema: {'text': [{'name': 'director'}, {'name': 'genre'}], 'numeric': [{'name': 'year'}, {'name': 'rating'}], 'tag': []}
```

## 创建自查询检索器

现在我们可以实例化我们的检索器了。为此，我们需要提供关于我们的文档支持的元数据字段的一些信息以及文档内容的简短描述。

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
```
```python
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
/Users/bagatur/langchain/libs/langchain/langchain/chains/llm.py:278: UserWarning: 预测和解析方法已被弃用，请直接将输出解析器传递给 LLMChain。
  warnings.warn(
```
```
查询='恐龙' 筛选=None 限制=None
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'id': 'doc:movie_reviews:7b5481d753bc4135851b66fa61def7fb', 'director': '史蒂文·斯皮尔伯格', '类型': '科幻', '年份': '1993', '评分': '7.7'}),
 Document(page_content='玩具活了起来，乐在其中', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': '约翰·拉塞特', '类型': '动画', '年份': '1995', '评分': '9.1'}),
 Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': '安德烈·塔可夫斯基', '类型': '科幻', '年份': '1979', '评分': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': '今敏', '类型': '科幻', '年份': '2006', '评分': '8.6'})]
# 这个示例只指定了一个筛选条件
retriever.invoke("我想看一部评分高于8.4的电影")
```
```
查询=' ' 筛选=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='评分', value=8.4) 限制=None
```
```
[Document(page_content='玩具活了起来，乐在其中', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': '约翰·拉塞特', '类型': '动画', '年份': '1995', '评分': '9.1'}),
 Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': '安德烈·塔可夫斯基', '类型': '科幻', '年份': '1979', '评分': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': '今敏', '类型': '科幻', '年份': '2006', '评分': '8.6'})]
# 这个示例指定了查询和筛选条件
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```
```
查询='女性' 筛选=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='导演', value='格蕾塔·葛韦格') 限制=None
```
```
[Document(page_content='一群普通身材的女性极具魅力，一些男人为她们倾心', metadata={'id': 'doc:movie_reviews:bb899807b93c442083fd45e75a4779d5', 'director': '格蕾塔·葛韦格', '类型': '戏剧', '年份': '2019', '评分': '8.3'})]
# 这个示例指定了复合筛选条件
retriever.invoke("有哪些评分高于8.5的科幻电影")
```
```
查询=' ' 筛选=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='评分', value=8.5), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='类型', value='科幻')]) 限制=None
```
```
[Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': '安德烈·塔可夫斯基', '类型': '科幻', '年份': '1979', '评分': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': '今敏', '类型': '科幻', '年份': '2006', '评分': '8.6'})]
# 这个示例指定了查询和复合筛选条件
retriever.invoke(
    "1990年后但2005年前关于玩具的电影，最好是动画片"
)
```
```
查询='玩具' 筛选=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='年份', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='年份', value=2005), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='类型', value='动画')]) 限制=None
```
```
[Document(page_content='玩具活了起来，乐在其中', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': '约翰·拉塞特', '类型': '动画', '年份': '1995', '评分': '9.1'})]
## 筛选 k
我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。
我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。
```
```
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```
```
# 这个示例只指定了一个相关查询
retriever.invoke("关于恐龙的两部电影是什么")
```
```
查询='恐龙' 筛选=None 限制=2
```
```

```

# 电影评论

## 《侏罗纪公园》

- 导演：Steven Spielberg

- 类型：科幻

- 年代：1993

- 评分：7.7

一群科学家复活了恐龙，结果引发了一系列混乱。

## 《玩具总动员》

- 导演：John Lasseter

- 类型：动画

- 年代：1995

- 评分：9.1

玩具们活了过来，并且玩得不亦乐乎。

```