# Supabase（Postgres）

[Supabase](https://supabase.com/docs) 是一个开源的 `Firebase` 替代品。`Supabase` 构建在 `PostgreSQL` 之上，提供强大的 `SQL` 查询功能，并能够与已有的工具和框架简单地进行接口交互。

[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) 也被称为 `Postgres`，是一个强调可扩展性和 `SQL` 兼容性的免费开源关系数据库管理系统（RDBMS）。

[Supabase](https://supabase.com/docs/guides/ai) 提供了一个开源工具包，用于开发使用 Postgres 和 pgvector 的 AI 应用程序。使用 Supabase 客户端库来存储、索引和查询您的向量嵌入以实现规模化。

在本文中，我们将演示围绕 `Supabase` 向量存储器的 `SelfQueryRetriever`。

具体来说，我们将：

1. 创建一个 Supabase 数据库

2. 启用 `pgvector` 扩展

3. 创建一个 `documents` 表和 `match_documents` 函数，这些将被 `SupabaseVectorStore` 使用

4. 将示例文档加载到向量存储器（数据库表）中

5. 构建并测试一个自查询检索器

## 设置 Supabase 数据库

1. 前往 https://database.new 创建您的 Supabase 数据库。

2. 在工作室中，转到[SQL编辑器](https://supabase.com/dashboard/project/_/sql/new)，运行以下脚本以启用 `pgvector` 并设置您的数据库作为向量存储器：

    ```sql
    -- 启用 pgvector 扩展以处理嵌入向量
    create extension if not exists vector;
    -- 创建一个表来存储您的文档
    create table
      documents (
        id uuid primary key,
        content text, -- 对应 Document.pageContent
        metadata jsonb, -- 对应 Document.metadata
        embedding vector (1536) -- 1536 适用于 OpenAI 嵌入，如有需要可更改
      );
    -- 创建一个用于搜索文档的函数
    create function match_documents (
      query_embedding vector (1536),
      filter jsonb default '{}'
    ) returns table (
      id uuid,
      content text,
      metadata jsonb,
      similarity float
    ) language plpgsql as $$
    #variable_conflict use_column
    begin
      return query
      select
        id,
        content,
        metadata,
        1 - (documents.embedding <=> query_embedding) as similarity
      from documents
      where metadata @> filter
      order by documents.embedding <=> query_embedding;
    end;
    $$;
    ```

## 创建 Supabase 向量存储器

接下来，我们将创建一个 Supabase 向量存储器，并使用一些数据进行填充。我们创建了一个包含电影摘要的小型演示文档集。

请确保使用以下命令安装具有 `openai` 支持的最新版本 `langchain`：

```python
%pip install --upgrade --quiet  langchain langchain-openai tiktoken
```

自查询检索器需要您安装 `lark`：

```python
%pip install --upgrade --quiet  lark
```

我们还需要 `supabase` 包：

```python
%pip install --upgrade --quiet  supabase
```

由于我们使用了 `SupabaseVectorStore` 和 `OpenAIEmbeddings`，我们必须加载它们的 API 密钥。

- 要找到您的 `SUPABASE_URL` 和 `SUPABASE_SERVICE_KEY`，请转到您的 Supabase 项目的[API设置](https://supabase.com/dashboard/project/_/settings/api)。

  - `SUPABASE_URL` 对应项目 URL

  - `SUPABASE_SERVICE_KEY` 对应 `service_role` API 密钥

- 要获取您的 `OPENAI_API_KEY`，请转到您的 OpenAI 帐户的[API密钥](https://platform.openai.com/account/api-keys)，并创建一个新的密钥。

```python
import getpass
import os
os.environ["SUPABASE_URL"] = getpass.getpass("Supabase URL:")
os.environ["SUPABASE_SERVICE_KEY"] = getpass.getpass("Supabase Service Key:")
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

_可选：_ 如果您将 Supabase 和 OpenAI API 密钥存储在 `.env` 文件中，您可以使用 [`dotenv`](https://github.com/theskumar/python-dotenv) 加载它们。

```python
%pip install --upgrade --quiet  python-dotenv
```

```python
from dotenv import load_dotenv
load_dotenv()
```

首先，我们将创建一个 Supabase 客户端并实例化一个 OpenAI 嵌入类。

```python
import os
from langchain_community.vectorstores import SupabaseVectorStore
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from supabase.client import Client, create_client
supabase_url = os.environ.get("SUPABASE_URL")
supabase_key = os.environ.get("SUPABASE_SERVICE_KEY")
supabase: Client = create_client(supabase_url, supabase_key)
embeddings = OpenAIEmbeddings()
```

接下来让我们创建我们的文档。

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
        page_content="一位心理学家/侦探迷失在一系列梦中的梦中的梦中，电影《盗梦空间》重新利用了这个概念",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通大小的女性非常纯真，一些男性对她们倾心",
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
vectorstore = SupabaseVectorStore.from_documents(
    docs,
    embeddings,
    client=supabase,
    table_name="documents",
    query_name="match_documents",
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提供一些关于我们的文档支持的元数据字段以及文档内容的简要描述。

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
        description="电影发行年份",
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
document_content_description = "电影的简要概述"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试

现在我们可以尝试实际使用我们的检索器！

```python
# 这个例子只指定了一个相关查询
retriever.invoke("有哪些关于恐龙的电影")
```

```output
query='dinosaur' filter=None limit=None
```

```output
[Document(page_content='一群科学家复活恐龙，混乱不断', metadata={'year': 1993, 'genre': '科幻', 'rating': 7.7}),
 Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': '科幻', 'rating': 9.9, 'director': '安德烈·塔可夫斯基'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，梦中梦中梦，而《盗梦空间》重新利用了这个概念', metadata={'year': 2006, 'rating': 8.6, 'director': '今敏'})]
```

```python
# 这个例子只指定了一个过滤器
retriever.invoke("我想看一部评分高于8.5的电影")
```

```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': '科幻', 'rating': 9.9, 'director': '安德烈·塔可夫斯基'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，梦中梦中梦，而《盗梦空间》重新利用了这个概念', metadata={'year': 2006, 'rating': 8.6, 'director': '今敏'})]
```

```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("格蕾塔·葛韦格导演过关于女性的电影吗")
```

```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```

```output
[Document(page_content='一群普通身材的女性极具魅力，一些男人为她们倾倒', metadata={'year': 2019, 'rating': 8.3, 'director': '格蕾塔·葛韦格'})]
```

```python
# 这个例子指定了一个复合过滤器
retriever.invoke("有哪些评分很高（超过8.5分）的科幻电影")
```

```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='science fiction')]) limit=None
```

```output
[Document(page_content='三个人走进区域，三个人走出区域', metadata={'year': 1979, 'genre': '科幻', 'rating': 9.9, 'director': '安德烈·塔可夫斯基'})]
```

```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "1990年后但在2005年之前（或之前）的关于玩具的电影，最好是动画片"
)
```

```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LTE: 'lte'>, attribute='year', value=2005), Comparison(comparator=<Comparator.LIKE: 'like'>, attribute='genre', value='animated')]) limit=None
```

```output
[Document(page_content='玩具活了起来，玩得不亦乐乎', metadata={'year': 1995, 'genre': '动画'})]
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
query='恐龙' filter=None limit=2
```

```output
[文档(page_content='一群科学家复活了恐龙，引发了混乱', metadata={'年份': 1993, '类型': '科幻', '评分': 7.7}),
 文档(page_content='玩具们活了起来，并且玩得不亦乐乎', metadata={'年份': 1995, '类型': '动画'})]
```