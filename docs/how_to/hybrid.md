# 混合搜索

LangChain中的标准搜索是通过向量相似性进行的。然而，许多向量存储实现（如Astra DB、ElasticSearch、Neo4J、AzureSearch等）还支持将向量相似性搜索与其他搜索技术（全文、BM25等）结合使用。这通常被称为“混合”搜索。

**步骤1：确保您使用的向量存储支持混合搜索**

目前，在LangChain中执行混合搜索没有统一的方法。每个向量存储可能有自己的方法来执行混合搜索。通常，这是通过在`similarity_search`期间传递的关键字参数来公开的。通过阅读文档或源代码，找出您使用的向量存储是否支持混合搜索，如果支持，如何使用它。

**步骤2：将该参数添加为链的可配置字段**

这将使您能够在运行时轻松调用链并配置任何相关的标志。有关配置的更多信息，请参见[此文档](/docs/how_to/configure)。

**步骤3：使用该可配置字段调用链**

现在，在运行时，您可以使用可配置字段调用此链。

## 代码示例

让我们看一个具体的代码示例，了解在代码中是什么样子。我们将使用Astra DB的Cassandra/CQL接口进行示例。

安装以下Python包：

```python
!pip install "cassio>=0.1.7"
```

获取[连接密钥](https://docs.datastax.com/en/astra/astra-db-vector/get-started/quickstart.html)。

初始化cassio：

```python
import cassio
cassio.init(
    database_id="您的数据库ID",
    token="您的应用程序令牌",
    keyspace="您的键空间",
)
```

使用标准的[index analyzer](https://docs.datastax.com/en/astra/astra-db-vector/cql/use-analyzers-with-cql.html)创建Cassandra VectorStore。索引分析器用于启用术语匹配。

```python
from cassio.table.cql import STANDARD_ANALYZER
from langchain_community.vectorstores import Cassandra
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
vectorstore = Cassandra(
    embedding=embeddings,
    table_name="test_hybrid",
    body_index_options=[STANDARD_ANALYZER],
    session=None,
    keyspace=None,
)
vectorstore.add_texts(
    [
        "In 2023, I visited Paris",
        "In 2022, I visited New York",
        "In 2021, I visited New Orleans",
    ]
)
```

如果我们进行标准的相似性搜索，我们会得到所有的文档：

```python
vectorstore.as_retriever().invoke("我最后访问的城市是哪个？")
```

```output
[Document(page_content='In 2022, I visited New York'),
Document(page_content='In 2023, I visited Paris'),
Document(page_content='In 2021, I visited New Orleans')]
```

Astra DB向量存储的`body_search`参数可用于过滤对术语`new`的搜索。

```python
vectorstore.as_retriever(search_kwargs={"body_search": "new"}).invoke(
    "我最后访问的城市是哪个？"
)
```

```output
[Document(page_content='In 2022, I visited New York'),
Document(page_content='In 2021, I visited New Orleans')]
```

现在，我们可以创建用于进行问答的链条。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import (
    ConfigurableField,
    RunnablePassthrough,
)
from langchain_openai import ChatOpenAI
```

这是基本的问答链条设置。

```python
template = """仅基于以下上下文回答问题：
{context}
问题：{question}
"""
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()
retriever = vectorstore.as_retriever()
```

在这里，我们将检索器标记为具有可配置字段。所有向量存储检索器都具有`search_kwargs`作为字段。这只是一个字典，其中包含向量存储特定的字段。

```python
configurable_retriever = retriever.configurable_fields(
    search_kwargs=ConfigurableField(
        id="search_kwargs",
        name="搜索参数",
        description="要使用的搜索参数",
    )
)
```

现在，我们可以使用可配置的检索器创建链条。

```python
chain = (
    {"context": configurable_retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```

```python
chain.invoke("我最后访问的城市是哪个？")
```

```output
巴黎
```

现在，我们可以使用可配置选项调用链条。`search_kwargs`是可配置字段的ID。值是要用于Astra DB的搜索参数。

```python
chain.invoke(
    "我最后访问的城市是哪个？",
    config={"configurable": {"search_kwargs": {"body_search": "new"}}},
)
```

```output
纽约
```