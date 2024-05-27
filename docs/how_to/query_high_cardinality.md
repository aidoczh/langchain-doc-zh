---

sidebar_position: 7

---

# 如何处理查询分析中的高基数分类变量

在进行查询分析时，您可能希望在分类列上创建一个过滤器。其中一个困难在于，通常需要指定确切的分类值。问题在于，您需要确保LLM准确生成该分类值。当只有少量有效值时，可以相对容易地通过提示来实现这一点。但是当有效值的数量很大时，情况就会变得更加困难，因为这些值可能不适合LLM的上下文，或者（如果适合）LLM可能无法正确处理其中的太多值。

在这篇笔记中，我们将看看如何解决这个问题。

## 设置

#### 安装依赖

```python
# %pip install -qU langchain langchain-community langchain-openai faker langchain-chroma
```

#### 设置环境变量

在本示例中，我们将使用OpenAI：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 可选项，取消注释以使用LangSmith跟踪运行。在此注册：https://smith.langchain.com。
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

#### 设置数据

我们将生成一堆假名字：

```python
from faker import Faker
fake = Faker()
names = [fake.name() for _ in range(10000)]
```

让我们看看其中一些名字：

```python
names[0]
```

输出：

```
'Hayley Gonzalez'
```

```python
names[567]
```

输出：

```
'Jesse Knight'
```

## 查询分析

现在我们可以设置一个基准查询分析：

```python
from langchain_core.pydantic_v1 import BaseModel, Field
```

```python
class Search(BaseModel):
    query: str
    author: str
```

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
system = """Generate a relevant search query for a library system"""
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

输出：

```
/Users/harrisonchase/workplace/langchain/libs/core/langchain_core/_api/beta_decorator.py:86: LangChainBetaWarning: The function `with_structured_output` is in beta. It is actively being worked on, so the API may change.
  warn_beta(
```

我们可以看到，如果我们拼写名字完全正确，它就知道如何处理：

```python
query_analyzer.invoke("what are books about aliens by Jesse Knight")
```

输出：

```
Search(query='books about aliens', author='Jesse Knight')
```

问题在于，您想要过滤的值可能并没有完全正确拼写：

```python
query_analyzer.invoke("what are books about aliens by jess knight")
```

输出：

```
Search(query='books about aliens', author='Jess Knight')
```

### 添加所有值

解决这个问题的一种方法是将所有可能的值添加到提示中。这通常会引导查询朝着正确的方向发展：

```python
system = """Generate a relevant search query for a library system.
`author` attribute MUST be one of:
{authors}
Do NOT hallucinate author name!"""
base_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
prompt = base_prompt.partial(authors=", ".join(names))
```

```python
query_analyzer_all = {"question": RunnablePassthrough()} | prompt | structured_llm
```

然而... 如果分类变量的列表足够长，可能会出错：

```python
try:
    res = query_analyzer_all.invoke("what are books about aliens by jess knight")
except Exception as e:
    print(e)
```

输出：

```
Error code: 400 - {'error': {'message': "This model's maximum context length is 16385 tokens. However, your messages resulted in 33885 tokens (33855 in the messages, 30 in the functions). Please reduce the length of the messages or functions.", 'type': 'invalid_request_error', 'param': 'messages', 'code': 'context_length_exceeded'}}
```

我们可以尝试使用更长的上下文窗口... 但是由于信息太多，它不一定能可靠地捕捉到：

```python
llm_long = ChatOpenAI(model="gpt-4-turbo-preview", temperature=0)
structured_llm_long = llm_long.with_structured_output(Search)
query_analyzer_all = {"question": RunnablePassthrough()} | prompt | structured_llm_long
```

```python
query_analyzer_all.invoke("what are books about aliens by jess knight")
```

输出：

```
Search(query='aliens', author='Kevin Knight')
```

### 查找并添加所有相关值

---

相反，我们可以对相关数值创建索引，然后查询前 N 个最相关的数值，

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(names, embeddings, collection_name="author_names")
```

```python
def select_names(question):
    _docs = vectorstore.similarity_search(question, k=10)
    _names = [d.page_content for d in _docs]
    return ", ".join(_names)
```

```python
create_prompt = {
    "question": RunnablePassthrough(),
    "authors": select_names,
} | base_prompt
```

```python
query_analyzer_select = create_prompt | structured_llm
```

```python
create_prompt.invoke("what are books by jess knight")
```

```output
ChatPromptValue(messages=[SystemMessage(content='Generate a relevant search query for a library system.\n\n`author` attribute MUST be one of:\n\nJesse Knight, Kelly Knight, Scott Knight, Richard Knight, Andrew Knight, Katherine Knight, Erica Knight, Ashley Knight, Becky Knight, Kevin Knight\n\nDo NOT hallucinate author name!'), HumanMessage(content='what are books by jess knight')])
```

```python
query_analyzer_select.invoke("what are books about aliens by jess knight")
```

```output
Search(query='books about aliens', author='Jesse Knight')
```

### 选择后替换

另一种方法是让 LLM 填充任何值，然后将该值转换为有效值。

实际上，这可以通过 Pydantic 类本身完成！

```python
from langchain_core.pydantic_v1 import validator
class Search(BaseModel):
    query: str
    author: str
    @validator("author")
    def double(cls, v: str) -> str:
        return vectorstore.similarity_search(v, k=1)[0].page_content
```

```python
system = """Generate a relevant search query for a library system"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
corrective_structure_llm = llm.with_structured_output(Search)
corrective_query_analyzer = (
    {"question": RunnablePassthrough()} | prompt | corrective_structure_llm
)
```

```python
corrective_query_analyzer.invoke("what are books about aliens by jes knight")
```

```output
Search(query='books about aliens', author='Jesse Knight')
```

```python
# TODO: show trigram similarity
```