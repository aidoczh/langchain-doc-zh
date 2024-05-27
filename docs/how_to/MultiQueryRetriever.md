# 如何使用 MultiQueryRetriever

基于距离的向量数据库检索将查询嵌入到高维空间中，并根据距离度量找到类似嵌入文档。但是，检索可能会因查询措辞的细微变化或嵌入未能很好地捕捉数据的语义而产生不同的结果。有时会进行提示工程/调整来手动解决这些问题，但可能会很繁琐。

[MultiQueryRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.multi_query.MultiQueryRetriever.html) 通过使用 LLM 自动化提示调整的过程，从不同角度生成多个查询，以针对给定的用户输入查询。对于每个查询，它检索一组相关文档，并对所有查询的唯一并集进行操作，以获得一个更大的潜在相关文档集。通过对同一问题生成多个视角，`MultiQueryRetriever` 可以缓解基于距离的检索的一些限制，并获得更丰富的结果。

让我们使用 Lilian Weng 的[RAG tutorial](/docs/tutorials/rag)中的[Lilian Weng 的《LLM Powered Autonomous Agents》](https://lilianweng.github.io/posts/2023-06-23-agent/)博客文章构建一个向量存储：

```python
# 构建一个示例向量数据库
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 加载博客文章
loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()
# 分割
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
splits = text_splitter.split_documents(data)
# 向量数据库
embedding = OpenAIEmbeddings()
vectordb = Chroma.from_documents(documents=splits, embedding=embedding)
```

#### 简单用法

指定用于生成查询的 LLM，检索器将完成其余工作。

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_openai import ChatOpenAI
question = "What are the approaches to Task Decomposition?"
llm = ChatOpenAI(temperature=0)
retriever_from_llm = MultiQueryRetriever.from_llm(
    retriever=vectordb.as_retriever(), llm=llm
)
```

```python
# 设置查询的日志记录
import logging
logging.basicConfig()
logging.getLogger("langchain.retrievers.multi_query").setLevel(logging.INFO)
```

```python
unique_docs = retriever_from_llm.invoke(question)
len(unique_docs)
```

```output
INFO:langchain.retrievers.multi_query:Generated queries: ['1. How can Task Decomposition be achieved through different methods?', '2. What strategies are commonly used for Task Decomposition?', '3. What are the various techniques for breaking down tasks in Task Decomposition?']
```

```output
5
```

请注意，检索器生成的基础查询被记录在 `INFO` 级别。

#### 提供自定义提示

在幕后，`MultiQueryRetriever` 使用特定的[prompt](https://api.python.langchain.com/en/latest/_modules/langchain/retrievers/multi_query.html#MultiQueryRetriever)生成查询。要自定义此提示：

1. 使用输入变量为问题创建[PromptTemplate](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.prompt.PromptTemplate.html)；

2. 实现一个[输出解析器](/docs/concepts#output-parsers)，如下面的解析器，将结果拆分为查询列表。

提示和输出解析器必须共同支持生成查询列表。

```python
from typing import List
from langchain_core.output_parsers import BaseOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
# 输出解析器将将 LLM 结果解析为行列表
class LineListOutputParser(BaseOutputParser[List[str]]):
    """用于行列表的输出解析器。"""
    def parse(self, text: str) -> List[str]:
        lines = text.strip().split("\n")
        return lines
output_parser = LineListOutputParser()
QUERY_PROMPT = PromptTemplate(
    input_variables=["question"],
    template="""You are an AI language model assistant. Your task is to generate five 
    different versions of the given user question to retrieve relevant documents from a vector 
    database. By generating multiple perspectives on the user question, your goal is to help
    the user overcome some of the limitations of the distance-based similarity search. 
    Provide these alternative questions separated by newlines.
    Original question: {question}""",
)
llm = ChatOpenAI(temperature=0)
# Chain
llm_chain = QUERY_PROMPT | llm | output_parser
# Other inputs
question = "What are the approaches to Task Decomposition?"
```

```python
# 运行
retriever = MultiQueryRetriever(
    retriever=vectordb.as_retriever(), llm_chain=llm_chain, parser_key="lines"
)  # "lines" 是解析输出的键（属性名称）
# 结果
unique_docs = retriever.invoke("What does the course say about regression?")
len(unique_docs)
```

```output
INFO:langchain.retrievers.multi_query:生成的查询：['1. 你能提供课程资料中关于回归的见解吗？', '2. 课程内容中如何讨论回归？', '3. 课程提供了关于回归的哪些信息？', '4. 课程中如何涵盖回归？', '5. 课程对回归有哪些教导？']
```

```output
9
```

