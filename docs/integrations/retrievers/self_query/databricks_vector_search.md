# Databricks 矢量搜索

[Databricks 矢量搜索](https://docs.databricks.com/en/generative-ai/vector-search.html) 是一个无服务器相似度搜索引擎，允许您在矢量数据库中存储数据的矢量表示，包括元数据。使用矢量搜索，您可以从由 Unity Catalog 管理的 Delta 表中创建自动更新的矢量搜索索引，并使用简单的 API 进行查询，以返回最相似的矢量。

在本教程中，我们将演示使用 Databricks 矢量搜索的 `SelfQueryRetriever`。

## 创建 Databricks 矢量存储索引

首先，我们需要创建一个 Databricks 矢量存储索引，并使用一些数据进行初始化。我们创建了一个包含电影摘要的小型演示文档集。

**注意：** 自查询检索器要求您安装了 `lark` (`pip install lark`) 以及特定集成的要求。

```python
%pip install --upgrade --quiet  langchain-core databricks-vectorsearch langchain-openai tiktoken
```

```output
注意：您可能需要重新启动内核以使用更新后的软件包。
```

我们希望使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
databricks_host = getpass.getpass("Databricks host:")
databricks_token = getpass.getpass("Databricks token:")
```

```output
OpenAI API Key: ········
Databricks host: ········
Databricks token: ········
```

```python
from databricks.vector_search.client import VectorSearchClient
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
emb_dim = len(embeddings.embed_query("hello"))
vector_search_endpoint_name = "vector_search_demo_endpoint"
vsc = VectorSearchClient(
    workspace_url=databricks_host, personal_access_token=databricks_token
)
vsc.create_endpoint(name=vector_search_endpoint_name, endpoint_type="STANDARD")
```

```output
[NOTICE] 使用个人身份验证令牌 (PAT)。仅建议用于开发。为了提高性能，请使用基于服务主体的身份验证。要禁用此消息，请将 disable_notice=True 传递给 VectorSearchClient()。
```

```python
index_name = "udhay_demo.10x.demo_index"
index = vsc.create_direct_access_index(
    endpoint_name=vector_search_endpoint_name,
    index_name=index_name,
    primary_key="id",
    embedding_dimension=emb_dim,
    embedding_vector_column="text_vector",
    schema={
        "id": "string",
        "page_content": "string",
        "year": "int",
        "rating": "float",
        "genre": "string",
        "text_vector": "array<float>",
    },
)
index.describe()
```

```python
index = vsc.get_index(endpoint_name=vector_search_endpoint_name, index_name=index_name)
index.describe()
```

```python
from langchain_core.documents import Document
docs = [
    Document(
        page_content="一群科学家复活了恐龙，然后一切都失控了",
        metadata={"id": 1, "year": 1993, "rating": 7.7, "genre": "动作"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，梦中的梦中迷失，一发不可收拾...",
        metadata={"id": 2, "year": 2010, "genre": "惊悚", "rating": 8.2},
    ),
    Document(
        page_content="一群普通身材的女性非常纯真，一些男性为她们倾心",
        metadata={"id": 3, "year": 2019, "rating": 8.3, "genre": "戏剧"},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={"id": 4, "year": 1979, "rating": 9.9, "genre": "科幻"},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，梦中的梦中的梦，而《盗梦空间》重复了这个想法",
        metadata={"id": 5, "year": 2006, "genre": "惊悚", "rating": 9.0},
    ),
    Document(
        page_content="玩具们活了过来，并且玩得不亦乐乎",
        metadata={"id": 6, "year": 1995, "genre": "动画", "rating": 9.3},
    ),
]
```

```python
from langchain_community.vectorstores import DatabricksVectorSearch
vector_store = DatabricksVectorSearch(
    index,
    text_column="page_content",
    embedding=embeddings,
    columns=["year", "rating", "genre"],
)
```

```python
vector_store.add_documents(docs)
```

## 创建我们的自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供一些关于我们的文档支持的元数据字段以及文档内容的简短描述。

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
        description="电影上映的年份",
        type="整数",
    ),
    AttributeInfo(
        name="rating", description="电影的评分（1-10）", type="浮点数"
    ),
]
document_content_description = "电影的简要概述"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vector_store, document_content_description, metadata_field_info, verbose=True
)
```

## 试一试

现在我们可以尝试使用我们的检索器了！

```python
# 这个例子只指定了一个相关的查询
retriever.invoke("有哪些关于恐龙的电影")
```

```output
[Document(page_content='一群科学家复活了恐龙，然后造成了混乱', metadata={'year': 1993.0, 'rating': 7.7, 'genre': '动作', 'id': 1.0}),
 Document(page_content='玩具活了起来，并且玩得很开心', metadata={'year': 1995.0, 'rating': 9.3, 'genre': '动画', 'id': 6.0}),
 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979.0, 'rating': 9.9, 'genre': '科幻', 'id': 4.0}),
 Document(page_content='一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', metadata={'year': 2006.0, 'rating': 9.0, 'genre': '惊悚', 'id': 5.0})]
```

```python
# 这个例子指定了一个过滤器
retriever.invoke("有哪些评分高于9的电影？")
```

```output
[Document(page_content='玩具活了起来，并且玩得很开心', metadata={'year': 1995.0, 'rating': 9.3, 'genre': '动画', 'id': 6.0}),
 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979.0, 'rating': 9.9, 'genre': '科幻', 'id': 4.0})]
```

```python
# 这个例子同时指定了一个相关的查询和一个过滤器
retriever.invoke("有哪些高评分的惊悚电影？")
```

```output
[Document(page_content='一位心理学家/侦探迷失在一连串的梦境中，而《盗梦空间》重复了这个想法', metadata={'year': 2006.0, 'rating': 9.0, 'genre': '惊悚', 'id': 5.0}),
 Document(page_content='莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中的梦中...', metadata={'year': 2010.0, 'rating': 8.2, 'genre': '惊悚', 'id': 2.0})]
```

```python
# 这个例子指定了一个查询和一个组合过滤器
retriever.invoke(
    "1990年后2005年前有关恐龙的电影，最好有很多动作"
)
```

```output
[Document(page_content='一群科学家复活了恐龙，然后造成了混乱', metadata={'year': 1993.0, 'rating': 7.7, 'genre': '动作', 'id': 1.0})]
```

## 过滤 k

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
retriever.invoke("有哪两部关于恐龙的电影？")
```