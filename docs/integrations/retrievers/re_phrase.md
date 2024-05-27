# RePhraseQuery

`RePhraseQuery` 是一个简单的检索器，它在用户输入和检索器传递的查询之间应用 LLM。

它可以用于以任何方式预处理用户输入。

## 示例

### 设置

创建一个向量存储。

```python
import logging
from langchain.retrievers import RePhraseQueryRetriever
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
logging.basicConfig()
logging.getLogger("langchain.retrievers.re_phraser").setLevel(logging.INFO)
loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)
vectorstore = Chroma.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())
```

### 使用默认提示

在 `from_llm` 类方法中使用的默认提示：

```
DEFAULT_TEMPLATE = """You are an assistant tasked with taking a natural language \
query from a user and converting it into a query for a vectorstore. \
In this process, you strip out information that is not relevant for \
the retrieval task. Here is the user query: {question}"""
```

```python
llm = ChatOpenAI(temperature=0)
retriever_from_llm = RePhraseQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(), llm=llm
)
```

```python
docs = retriever_from_llm.invoke(
    "Hi I'm Lance. What are the approaches to Task Decomposition?"
)
```

```output
INFO:langchain.retrievers.re_phraser:Re-phrased question: The user query can be converted into a query for a vectorstore as follows:
"approaches to Task Decomposition"
```

```python
docs = retriever_from_llm.invoke(
    "I live in San Francisco. What are the Types of Memory?"
)
```

```output
INFO:langchain.retrievers.re_phraser:Re-phrased question: Query for vectorstore: "Types of Memory"
```

### 自定义提示

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
QUERY_PROMPT = PromptTemplate(
    input_variables=["question"],
    template="""You are an assistant tasked with taking a natural languge query from a user
    and converting it into a query for a vectorstore. In the process, strip out all 
    information that is not relevant for the retrieval task and return a new, simplified
    question for vectorstore retrieval. The new user query should be in pirate speech.
    Here is the user query: {question} """,
)
llm = ChatOpenAI(temperature=0)
llm_chain = LLMChain(llm=llm, prompt=QUERY_PROMPT)
```

```python
retriever_from_llm_chain = RePhraseQueryRetriever(
    retriever=vectorstore.as_retriever(), llm_chain=llm_chain
)
```

```python
docs = retriever_from_llm_chain.invoke(
    "Hi I'm Lance. What is Maximum Inner Product Search?"
)
```

```output
INFO:langchain.retrievers.re_phraser:Re-phrased question: Ahoy matey! What be Maximum Inner Product Search, ye scurvy dog?
```