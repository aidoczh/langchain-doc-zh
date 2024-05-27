# 腾讯云 VectorDB

> [腾讯云 VectorDB](https://cloud.tencent.com/document/product/1709) 是一种完全托管、自主开发的企业级分布式数据库服务，专为存储、检索和分析多维向量数据而设计。

在本教程中，我们将使用腾讯云 VectorDB 演示 `SelfQueryRetriever`。

## 创建 TencentVectorDB 实例

首先，我们需要创建一个 TencentVectorDB 实例，并使用一些数据进行初始化。我们已经创建了一个小型演示数据集，其中包含电影摘要。

**注意：** self-query retriever 需要您安装 `lark`（`pip install lark`）以及特定于集成的要求。

```python
%pip install --upgrade --quiet tcvectordb langchain-openai tiktoken lark
```

```output
[notice] A new release of pip is available: 23.2.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

创建 TencentVectorDB 实例并使用一些数据进行初始化：

```python
from langchain_community.vectorstores.tencentvectordb import (
    ConnectionParams,
    MetaField,
    TencentVectorDB,
)
from langchain_core.documents import Document
from tcvectordb.model.enum import FieldType
meta_fields = [
    MetaField(name="year", data_type="uint64", index=True),
    MetaField(name="rating", data_type="string", index=False),
    MetaField(name="genre", data_type=FieldType.String, index=True),
    MetaField(name="director", data_type=FieldType.String, index=True),
]
docs = [
    Document(
        page_content="The Shawshank Redemption is a 1994 American drama film written and directed by Frank Darabont.",
        metadata={
            "year": 1994,
            "rating": "9.3",
            "genre": "drama",
            "director": "Frank Darabont",
        },
    ),
    Document(
        page_content="The Godfather is a 1972 American crime film directed by Francis Ford Coppola.",
        metadata={
            "year": 1972,
            "rating": "9.2",
            "genre": "crime",
            "director": "Francis Ford Coppola",
        },
    ),
    Document(
        page_content="The Dark Knight is a 2008 superhero film directed by Christopher Nolan.",
        metadata={
            "year": 2008,
            "rating": "9.0",
            "genre": "science fiction",
            "director": "Christopher Nolan",
        },
    ),
    Document(
        page_content="Inception is a 2010 science fiction action film written and directed by Christopher Nolan.",
        metadata={
            "year": 2010,
            "rating": "8.8",
            "genre": "science fiction",
            "director": "Christopher Nolan",
        },
    ),
    Document(
        page_content="The Avengers is a 2012 American superhero film based on the Marvel Comics superhero team of the same name.",
        metadata={
            "year": 2012,
            "rating": "8.0",
            "genre": "science fiction",
            "director": "Joss Whedon",
        },
    ),
    Document(
        page_content="Black Panther is a 2018 American superhero film based on the Marvel Comics character of the same name.",
        metadata={
            "year": 2018,
            "rating": "7.3",
            "genre": "science fiction",
            "director": "Ryan Coogler",
        },
    ),
]
vector_db = TencentVectorDB.from_documents(
    docs,
    None,
    connection_params=ConnectionParams(
        url="http://10.0.X.X",
        key="eC4bLRy2va******************************",
        username="root",
        timeout=20,
    ),
    collection_name="self_query_movies",
    meta_fields=meta_fields,
    drop_old=True,
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器了。为此，我们需要提供一些关于我们的文档支持的元数据字段的信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import ChatOpenAI
metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型",
        type="string",
    ),
    AttributeInfo(
        name="year",
        description="电影的上映年份",
        type="integer",
    ),
    AttributeInfo(
        name="director",
        description="电影导演的姓名",
        type="string",
    ),
    AttributeInfo(
        name="rating", description="电影的评分（1-10）", type="string"
    ),
]
document_content_description = "电影的简要摘要"
```

```python
llm = ChatOpenAI(temperature=0, model="gpt-4", max_tokens=4069)
retriever = SelfQueryRetriever.from_llm(
    llm, vector_db, document_content_description, metadata_field_info, verbose=True
)
```

## 试一试

现在我们可以尝试实际使用我们的检索器了！

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("关于超级英雄的电影")
```

```output
[Document(page_content='《蝙蝠侠：黑暗骑士》是一部2008年由克里斯托弗·诺兰执导的超级英雄电影。', metadata={'year': 2008, 'rating': '9.0', 'genre': '科幻', 'director': '克里斯托弗·诺兰'}),
 Document(page_content='《复仇者联盟》是一部2012年根据漫威漫画同名超级英雄团队改编的美国超级英雄电影。', metadata={'year': 2012, 'rating': '8.0', 'genre': '科幻', 'director': '乔斯·韦登'}),
 Document(page_content='《黑豹》是一部2018年根据漫威漫画同名角色改编的美国超级英雄电影。', metadata={'year': 2018, 'rating': '7.3', 'genre': '科幻', 'director': '瑞恩·库格勒'}),
 Document(page_content='《教父》是一部1972年由弗朗西斯·福特·科波拉执导的美国犯罪电影。', metadata={'year': 1972, 'rating': '9.2', 'genre': '犯罪', 'director': '弗朗西斯·福特·科波拉'})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("2010年后上映的电影")
```

```output
[Document(page_content='《复仇者联盟》是一部2012年根据漫威漫画同名超级英雄团队改编的美国超级英雄电影。', metadata={'year': 2012, 'rating': '8.0', 'genre': '科幻', 'director': '乔斯·韦登'}),
 Document(page_content='《黑豹》是一部2018年根据漫威漫画同名角色改编的美国超级英雄电影。', metadata={'year': 2018, 'rating': '7.3', 'genre': '科幻', 'director': '瑞恩·库格勒'})]
```

```python
# 这个例子同时指定了一个相关的查询和一个过滤器
retriever.invoke("2010年后上映的超级英雄电影")
```

```output
[Document(page_content='《复仇者联盟》是一部2012年根据漫威漫画同名超级英雄团队改编的美国超级英雄电影。', metadata={'year': 2012, 'rating': '8.0', 'genre': '科幻', 'director': '乔斯·韦登'}),
 Document(page_content='《黑豹》是一部2018年根据漫威漫画同名角色改编的美国超级英雄电影。', metadata={'year': 2018, 'rating': '7.3', 'genre': '科幻', 'director': '瑞恩·库格勒'})]
```

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vector_db,
    document_content_description,
    metadata_field_info,
    verbose=True,
    enable_limit=True,
)
```

```python
retriever.invoke("两部关于超级英雄的电影")
```

```output
[Document(page_content='《蝙蝠侠：黑暗骑士》是一部2008年由克里斯托弗·诺兰执导的超级英雄电影。', metadata={'year': 2008, 'rating': '9.0', 'genre': '科幻', 'director': '克里斯托弗·诺兰'}),
 Document(page_content='《复仇者联盟》是一部2012年根据漫威漫画同名超级英雄团队改编的美国超级英雄电影。', metadata={'year': 2012, 'rating': '8.0', 'genre': '科幻', 'director': '乔斯·韦登'})]
```