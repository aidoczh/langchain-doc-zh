# Vectara 

[Vectara](https://vectara.com/) 是一款值得信赖的 GenAI 平台，为文档索引和查询提供了易于使用的 API。

`Vectara` 提供了一个端到端的托管服务，用于 `检索增强生成` 或 [RAG](https://vectara.com/grounded-generation/)，其中包括：

1. 从文档文件中提取文本并将其分块成句子的方法。

2. 最先进的 [Boomerang](https://vectara.com/how-boomerang-takes-retrieval-augmented-generation-to-the-next-level-via-grounded-generation/) 嵌入模型。每个文本块都使用 `Boomerang` 编码为向量嵌入，并存储在 Vectara 内部知识（向量+文本）存储中。

3. 一个查询服务，自动将查询编码为嵌入，并检索出最相关的文本段落（包括对 [Hybrid Search](https://docs.vectara.com/docs/api-reference/search-apis/lexical-matching) 和 [MMR](https://vectara.com/get-diverse-results-and-comprehensive-summaries-with-vectaras-mmr-reranker/) 的支持）。

4. 根据检索到的文档创建 [生成式摘要](https://docs.vectara.com/docs/learn/grounded-generation/grounded-generation-overview)，包括引用。

查看 [Vectara API 文档](https://docs.vectara.com/docs/) 以获取有关如何使用 API 的更多信息。

这个笔记本展示了如何在 Vectara 中使用 `SelfQueryRetriever`。

# 设置

您需要一个 `Vectara` 帐户才能在 `LangChain` 中使用 `Vectara`。要开始，请按照以下步骤操作（请参阅我们的 [快速入门](https://docs.vectara.com/docs/quickstart) 指南）：

1. 如果您还没有 `Vectara` 帐户，请 [注册](https://console.vectara.com/signup) 一个。完成注册后，您将获得一个 Vectara 客户 ID。您可以通过单击 Vectara 控制台窗口右上角的您的名称来找到您的客户 ID。

2. 在您的帐户中，您可以创建一个或多个语料库。每个语料库代表一个区域，用于存储从输入文档摄取的文本数据。要创建语料库，请使用 **"创建语料库"** 按钮。然后为您的语料库提供一个名称和描述。您还可以选择定义过滤属性并应用一些高级选项。如果单击您创建的语料库，您可以在顶部看到其名称和语料库 ID。

3. 接下来，您需要创建 API 密钥以访问语料库。在语料库视图中点击 **"授权"** 选项卡，然后点击 **"创建 API 密钥"** 按钮。为您的密钥命名，并选择您想要的查询权限或查询+索引权限。点击 **"创建"**，现在您有一个活动的 API 密钥。请保密此密钥。

要在 LangChain 中使用 Vectara，您需要三个值：客户 ID、语料库 ID 和 API 密钥。

您可以通过以下两种方式将它们提供给 LangChain：

1. 在环境中包含这三个变量：`VECTARA_CUSTOMER_ID`、`VECTARA_CORPUS_ID` 和 `VECTARA_API_KEY`。

> 例如，您可以使用 `os.environ` 和 `getpass` 设置这些变量，如下所示：

```python
import os
import getpass
os.environ["VECTARA_CUSTOMER_ID"] = getpass.getpass("Vectara Customer ID:")
os.environ["VECTARA_CORPUS_ID"] = getpass.getpass("Vectara Corpus ID:")
os.environ["VECTARA_API_KEY"] = getpass.getpass("Vectara API Key:")
```

2. 在创建 `Vectara` vectorstore 对象时，将它们作为参数提供：

```python
vectorstore = Vectara(
                vectara_customer_id=vectara_customer_id,
                vectara_corpus_id=vectara_corpus_id,
                vectara_api_key=vectara_api_key
            )
```

**注意：** 自查询检索器需要您安装 `lark` (`pip install lark`)。

## 从 LangChain 连接到 Vectara

在这个示例中，我们假设您已经创建了一个帐户和一个语料库，并将您的 VECTARA_CUSTOMER_ID、VECTARA_CORPUS_ID 和 VECTARA_API_KEY（具有索引和查询权限的密钥）添加为环境变量。

语料库定义了 4 个字段作为用于过滤的元数据：年份、导演、评分和流派

```python
from langchain.chains import ConversationalRetrievalChain
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings import FakeEmbeddings
from langchain_community.vectorstores import Vectara
from langchain_core.documents import Document
from langchain_openai import OpenAI
from langchain_text_splitters import CharacterTextSplitter
```

```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后引发了混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦中的梦中的梦中，而《盗梦空间》重复了这个想法",
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
            "rating": 9.9,
            "director": "安德烈·塔可夫斯基",
            "genre": "科幻",
        },
    ),
]
vectara = Vectara()
for doc in docs:
    vectara.add_texts(
        [doc.page_content],
        embedding=FakeEmbeddings(size=768),
        doc_metadata=doc.metadata,
    )
```

## 创建自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简要描述。

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
document_content_description = "电影的简要概述"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectara, document_content_description, metadata_field_info, verbose=True
)
```

## 测试

现在我们可以尝试实际使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有关恐龙的一些电影")
```

```output
[Document(page_content='一群科学家复活了恐龙，然后混乱就开始了', metadata={'lang': 'eng', 'offset': '0', 'len': '66', 'year': '1993', 'rating': '7.7', 'genre': 'science fiction', 'source': 'langchain'}),
 Document(page_content='玩具活了起来，然后玩得不亦乐乎', metadata={'lang': 'eng', 'offset': '0', 'len': '41', 'year': '1995', 'genre': 'animated', 'source': 'langchain'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'lang': 'eng', 'offset': '0', 'len': '116', 'year': '2006', 'director': 'Satoshi Kon', 'rating': '8.6', 'source': 'langchain'}),
 Document(page_content='莱昂纳多迷失在一个梦中的梦中的梦中...', metadata={'lang': 'eng', 'offset': '0', 'len': '76', 'year': '2010', 'director': 'Christopher Nolan', 'rating': '8.2', 'source': 'langchain'}),
 Document(page_content='一群普通身材的女性非常健康，一些男人追求她们', metadata={'lang': 'eng', 'offset': '0', 'len': '82', 'year': '2019', 'director': 'Greta Gerwig', 'rating': '8.3', 'source': 'langchain'}),
 Document(page_content='三个人走进这个区域，三个人走出这个区域', metadata={'lang': 'eng', 'offset': '0', 'len': '60', 'year': '1979', 'rating': '9.9', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'source': 'langchain'})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
[Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重新利用了这个想法', metadata={'lang': 'eng', 'offset': '0', 'len': '116', 'year': '2006', 'director': 'Satoshi Kon', 'rating': '8.6', 'source': 'langchain'}),
 Document(page_content='三个人走进这个区域，三个人走出这个区域', metadata={'lang': 'eng', 'offset': '0', 'len': '60', 'year': '1979', 'rating': '9.9', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'source': 'langchain'})]
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
[Document(page_content='一群普通身材的女性非常健康，一些男人追求她们', metadata={'lang': 'eng', 'offset': '0', 'len': '82', 'year': '2019', 'director': 'Greta Gerwig', 'rating': '8.3', 'source': 'langchain'})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分很高（超过8.5）的科幻电影？")
```

```output
[Document(page_content='三个人走进这个区域，三个人走出这个区域', metadata={'lang': 'eng', 'offset': '0', 'len': '60', 'year': '1979', 'rating': '9.9', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'source': 'langchain'})]
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但2005年前的关于玩具的电影，最好是动画片"
)
```

```output
[Document(page_content='玩具活了起来，然后玩得不亦乐乎', metadata={'lang': 'eng', 'offset': '0', 'len': '41', 'year': '1995', 'genre': 'animated', 'source': 'langchain'})]
```

## 过滤器 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectara,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```

以上代码创建了一个 `SelfQueryRetriever` 对象，该对象使用了一个 `llm` 对象和一个 `vectara` 对象作为参数。`document_content_description` 和 `metadata_field_info` 是用于描述文档内容和元数据字段信息的参数。`enable_limit` 参数设置为 `True`，表示启用结果数量限制。`verbose` 参数设置为 `True`，表示在运行过程中输出详细信息。

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("关于恐龙的两部电影是什么")
```

```output
[Document(page_content='一群科学家复活恐龙，结果引发了混乱', metadata={'lang': 'eng', 'offset': '0', 'len': '66', 'year': '1993', 'rating': '7.7', 'genre': 'science fiction', 'source': 'langchain'}),
 Document(page_content='玩具活了过来，并且玩得不亦乐乎', metadata={'lang': 'eng', 'offset': '0', 'len': '41', 'year': '1995', 'genre': 'animated', 'source': 'langchain'})]
```