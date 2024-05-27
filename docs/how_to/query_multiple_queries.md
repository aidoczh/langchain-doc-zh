---

sidebar_position: 4

---

# 如何处理查询分析中的多个查询

有时，查询分析技术可能会生成多个查询。在这种情况下，我们需要记得运行所有查询，然后将结果合并。我们将展示一个简单示例（使用模拟数据）来说明如何做到这一点。

## 设置

#### 安装依赖

```python
# %pip install -qU langchain langchain-community langchain-openai langchain-chroma
```

#### 设置环境变量

在本示例中，我们将使用 OpenAI：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 可选，取消注释以使用 LangSmith 跟踪运行。在此注册：https://smith.langchain.com.
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 创建索引

我们将在虚构信息上创建一个向量存储。

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
texts = ["Harrison worked at Kensho", "Ankush worked at Facebook"]
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(
    texts,
    embeddings,
)
retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
```

## 查询分析

我们将使用函数调用来构造输出。让它返回多个查询。

```python
from typing import List, Optional
from langchain_core.pydantic_v1 import BaseModel, Field
class Search(BaseModel):
    """在工作记录数据库上进行搜索。"""
    queries: List[str] = Field(
        ...,
        description="要搜索的不同查询",
    )
```

```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
output_parser = PydanticToolsParser(tools=[Search])
system = """您可以发出搜索查询以获取信息以帮助回答用户信息。
如果您需要查找两个不同的信息片段，您是被允许的！"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```

```output
/Users/harrisonchase/workplace/langchain/libs/core/langchain_core/_api/beta_decorator.py:86: LangChainBetaWarning: 函数 `with_structured_output` 处于 beta 版本。正在积极开发中，因此 API 可能会更改。
  warn_beta(
```

我们可以看到这允许创建多个查询。

```python
query_analyzer.invoke("Harrison在哪里工作")
```

```output
Search(queries=['Harrison work location'])
```

```python
query_analyzer.invoke("Harrison和Ankush在哪里工作")
```

```output
Search(queries=['Harrison work place', 'Ankush work place'])
```

## 查询分析的检索

那么我们如何将这个包含在链中？使这变得更容易的一件事是，如果我们异步调用我们的检索器 - 这将使我们能够循环查询而不会在响应时间上被阻塞。

```python
from langchain_core.runnables import chain
```

```python
@chain
async def custom_chain(question):
    response = await query_analyzer.ainvoke(question)
    docs = []
    for query in response.queries:
        new_docs = await retriever.ainvoke(query)
        docs.extend(new_docs)
    # 您可能想考虑在这里重新排名或去重文档
    # 但这是一个单独的话题
    return docs
```

```python
await custom_chain.ainvoke("Harrison在哪里工作")
```

```output
[Document(page_content='Harrison worked at Kensho')]
```

```python
await custom_chain.ainvoke("Harrison和Ankush在哪里工作")
```

```output
[Document(page_content='Harrison worked at Kensho'),
 Document(page_content='Ankush worked at Facebook')]
```