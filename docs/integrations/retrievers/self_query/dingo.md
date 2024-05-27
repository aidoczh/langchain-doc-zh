# DingoDB

[DingoDB](https://dingodb.readthedocs.io/en/latest/) 是一个分布式的多模式向量数据库，结合了数据湖和向量数据库的特点，可以存储任何类型和大小的数据（键值、PDF、音频、视频等）。它具有实时低延迟的处理能力，实现快速的洞察和响应，并能高效地进行即时分析和处理多模态数据。

在本教程中，我们将使用 `DingoDB` 向量存储演示 `SelfQueryRetriever`。

## 创建 DingoDB 索引

首先，我们需要创建一个 `DingoDB` 向量存储，并用一些数据填充它。我们创建了一个包含电影摘要的小型演示文档集。

要使用 DingoDB，您应该已经运行了一个[DingoDB 实例](https://github.com/dingodb/dingo-deploy/blob/main/README.md)。

**注意：** self-query 检索器需要您安装了 `lark` 包。

```python
%pip install --upgrade --quiet  dingodb
# 或者安装最新版本：
%pip install --upgrade --quiet  git+https://github.com/dingodb/pydingo.git
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import os
OPENAI_API_KEY = ""
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain_community.vectorstores import Dingo
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
# 创建新索引
from dingodb import DingoDB
index_name = "langchain_demo"
dingo_client = DingoDB(user="", password="", host=["172.30.14.221:13000"])
# 首先，检查我们的索引是否已经存在。如果不存在，我们就创建它
if (
    index_name not in dingo_client.get_index()
    and index_name.upper() not in dingo_client.get_index()
):
    # 创建一个新索引，根据您自己的情况进行修改
    dingo_client.create_index(
        index_name=index_name, dimension=1536, metric_type="cosine", auto_id=False
    )
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后造成了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": '"动作", "科幻"'},
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
        page_content="一群普通身材的女人非常纯真，一些男人对她们垂涎三尺",
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
            "genre": '"科幻", "惊悚"',
            "rating": 9.9,
        },
    ),
]
vectorstore = Dingo.from_documents(
    docs, embeddings, index_name=index_name, client=dingo_client
)
```

```python
dingo_client.get_index()
dingo_client.delete_index("langchain_demo")
```

```output
True
```

```python
dingo_client.vector_count("langchain_demo")
```

```output
9
```

## 创建自查询检索器

现在我们可以实例化我们的检索器了。为此，我们需要提供一些关于文档支持的元数据字段和文档内容的简要描述的信息。

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
retriever.invoke("有关恐龙的一些电影")
```

```output
query='dinosaurs' filter=None limit=None
```

```output
[文档(page_content='一群科学家复活了恐龙，引发了混乱', metadata={'id': 1183188982475, 'text': '一群科学家复活了恐龙，引发了混乱', 'score': 0.13397777, 'year': {'value': 1993}, 'rating': {'value': 7.7}, 'genre': '"动作", "科幻"'}),
 文档(page_content='玩具活了起来，并且玩得很开心', metadata={'id': 1183189196391, 'text': '玩具活了起来，并且玩得很开心', 'score': 0.18994397, 'year': {'value': 1995}, 'genre': '动画'}),
 文档(page_content='三个人走进区域，三个人走出区域', metadata={'id': 1183189220159, 'text': '三个人走进区域，三个人走出区域', 'score': 0.23288351, 'year': {'value': 1979}, 'director': '安德烈·塔可夫斯基', 'rating': {'value': 9.9}, 'genre': '"科幻", "惊悚"'}),
 文档(page_content='一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重新利用了这个概念', metadata={'id': 1183189148854, 'text': '一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重新利用了这个概念', 'score': 0.24421334, 'year': {'value': 2006}, 'director': '今敏', 'rating': {'value': 8.6}})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
comparator=<Comparator.GT: 'gt'> attribute='rating' value=8.5
```

```output
[Document(page_content='三个人走进地带，三个人走出地带', metadata={'id': 1183189220159, 'text': '三个人走进地带，三个人走出地带', 'score': 0.25033575, 'year': {'value': 1979}, 'director': 'Andrei Tarkovsky', 'genre': '"science fiction", "thriller"', 'rating': {'value': 9.9}}),
 Document(page_content='一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', metadata={'id': 1183189148854, 'text': '一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', 'score': 0.26431882, 'year': {'value': 2006}, 'director': 'Satoshi Kon', 'rating': {'value': 8.6}})]
```

```python
# 这个例子指定了查询和过滤器
retriever.invoke("Greta Gerwig导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
comparator=<Comparator.EQ: 'eq'> attribute='director' value='Greta Gerwig'
```

```output
[Document(page_content='一群正常身材的女性非常健康，一些男人渴望她们', metadata={'id': 1183189172623, 'text': '一群正常身材的女性非常健康，一些男人渴望她们', 'score': 0.19482517, 'year': {'value': 2019}, 'director': 'Greta Gerwig', 'rating': {'value': 8.3}})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有没有一部评分很高（超过8.5）的科幻电影？")
```

```output
query='science fiction' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
comparator=<Comparator.GT: 'gt'> attribute='rating' value=8.5
```

```output
[Document(page_content='一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', metadata={'id': 1183189148854, 'text': '一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', 'score': 0.19805312, 'year': {'value': 2006}, 'director': 'Satoshi Kon', 'rating': {'value': 8.6}}),
 Document(page_content='三个人走进地带，三个人走出地带', metadata={'id': 1183189220159, 'text': '三个人走进地带，三个人走出地带', 'score': 0.225586, 'year': {'value': 1979}, 'director': 'Andrei Tarkovsky', 'rating': {'value': 9.9}, 'genre': '"science fiction", "thriller"'})]
```

```python
# 这个例子指定了查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前的关于玩具的电影，最好是动画片"
)
```

```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005)]), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated')]) limit=None
operator=<Operator.AND: 'and'> arguments=[Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005)]), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated')]
```

```output
[Document(page_content='玩具们活了起来，玩得很开心', metadata={'id': 1183189196391, 'text': '玩具们活了起来，玩得很开心', 'score': 0.133829, 'year': {'value': 1995}, 'genre': 'animated'})]
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
retriever.invoke("有关恐龙的两部电影是什么")
```

```output
query='dinosaurs' filter=None limit=2
```

```markdown
[文档(page_content='一群科学家复活恐龙，混乱不堪', metadata={'id': 1183188982475, 'text': '一群科学家复活恐龙，混乱不堪', 'score': 0.13394928, 'year': {'value': 1993}, 'rating': {'value': 7.7}, 'genre': '"action", "science fiction"'}),
 文档(page_content='玩具活了起来，玩得不亦乐乎', metadata={'id': 1183189196391, 'text': '玩具活了起来，玩得不亦乐乎', 'score': 0.1899159, 'year': {'value': 1995}, 'genre': 'animated'})]
```

### 自动语音识别（ASR）

自动语音识别（ASR）是指计算机识别和理解人类语音的过程。ASR系统接收音频输入，然后将其转换为文本形式。这种技术在语音助手（如Amazon Alexa、Apple Siri和Microsoft Cortana）和语音转录应用程序中得到广泛应用。

ASR系统的性能通常通过诸如**词错误率（Word Error Rate, WER）**这样的指标来衡量。WER是ASR输出与参考文本之间的不匹配词数的比率。例如，如果参考文本是“how are you”，而ASR系统输出“how old you”，那么WER为1/3。

ASR系统的发展离不开深度学习技术，如**循环神经网络（Recurrent Neural Networks, RNNs）**和**长短时记忆网络（Long Short-Term Memory, LSTM）**。这些技术有助于提高ASR系统对语音信号的理解能力，从而提高识别准确率。

除了常见的音频格式（如**Wave**和**FLAC**）外，ASR系统还可以处理其他格式，如**MPEG**和**JPEG**。这使得ASR系统可以应用于各种不同类型的语音数据。

如果想深入了解ASR系统的工作原理和最新研究进展，可以阅读以下论文：

- **[20]** Mohamed, A. R., Dahl, G. E., & Hinton, G. E. (2012). Acoustic modeling using deep belief networks. IEEE Transactions on Audio, Speech, and Language Processing, 20(1), 14-22.

希望这些信息能帮助您更好地理解自动语音识别技术的基本概念和应用。如果您有任何疑问，请随时向我提问。