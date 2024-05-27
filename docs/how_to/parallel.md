# 如何并行调用可运行对象

:::info 前提条件

本指南假定您熟悉以下概念：

- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)

- [运行可运行对象的链式结构](/docs/how_to/sequence)

:::

[`RunnableParallel`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html) 基本上是一个字典，其值是可运行对象（或可以强制转换为可运行对象的内容，比如函数）。它会并行运行所有值，并使用 `RunnableParallel` 的整体输入调用每个值。最终返回值是一个字典，其中包含每个值的结果及其相应的键。

## 使用 `RunnableParallel` 进行格式化

`RunnableParallel` 用于并行操作，也可用于调整一个可运行对象的输出，以匹配序列中下一个可运行对象的输入格式。您可以使用它们来分割或分叉链式结构，以便多个组件可以并行处理输入。随后，其他组件可以合并结果，以合成最终响应。这种类型的链式结构创建的计算图如下所示：

```text
     输入
      / \
     /   \
 分支1  分支2
     \   /
      \ /
      合并
```

下面，提示的输入预期为带有键 `"context"` 和 `"question"` 的映射。用户输入只是问题。因此，我们需要使用我们的检索器获取上下文，并将用户输入传递到 `"question"` 键下。

```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()
template = """Answer the question based only on the following context:
{context}
Question: {question}
"""
# 提示期望带有 "context" 和 "question" 键的输入
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()
retrieval_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
retrieval_chain.invoke("where did harrison work?")
```

```output
'Harrison worked at Kensho.'
```

::: {.callout-tip}

请注意，当将 `RunnableParallel` 与另一个可运行对象组合时，我们甚至不需要将字典包装在 `RunnableParallel` 类中 — 类型转换会为我们处理。在链式结构的上下文中，以下操作是等效的：

:::

```
{"context": retriever, "question": RunnablePassthrough()}
```

```
RunnableParallel({"context": retriever, "question": RunnablePassthrough()})
```

```
RunnableParallel(context=retriever, question=RunnablePassthrough())
```

有关更多信息，请参阅[强制转换部分](/docs/how_to/sequence/#coercion)。

## 使用 `itemgetter` 作为简写

请注意，您可以使用 Python 的 `itemgetter` 作为简写，从映射中提取数据，然后与 `RunnableParallel` 结合。您可以在[Python 文档](https://docs.python.org/3/library/operator.html#operator.itemgetter)中找到有关 `itemgetter` 的更多信息。

在下面的示例中，我们使用 `itemgetter` 从映射中提取特定键：

```python
from operator import itemgetter
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()
template = """Answer the question based only on the following context:
{context}
Question: {question}
Answer in the following language: {language}
"""
prompt = ChatPromptTemplate.from_template(template)
chain = (
    {
        "context": itemgetter("question") | retriever,
        "question": itemgetter("question"),
        "language": itemgetter("language"),
    }
    | prompt
    | model
    | StrOutputParser()
)
chain.invoke({"question": "where did harrison work", "language": "italian"})
```

```output
'Harrison ha lavorato a Kensho.'
```

## 并行化步骤

`RunnableParallel` 使得轻松执行多个可运行对象并行，并将这些可运行对象的输出作为映射返回。

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel
from langchain_openai import ChatOpenAI
model = ChatOpenAI()
joke_chain = ChatPromptTemplate.from_template("tell me a joke about {topic}") | model
poem_chain = (
    ChatPromptTemplate.from_template("write a 2-line poem about {topic}") | model
)
map_chain = RunnableParallel(joke=joke_chain, poem=poem_chain)
map_chain.invoke({"topic": "bear"})
```

## 并行处理

`RunnableParallel` 对于并行运行独立进程也非常有用，因为映射中的每个 `Runnable` 都是并行执行的。例如，我们可以看到我们之前的 `joke_chain`、`poem_chain` 和 `map_chain` 大致具有相同的运行时间，即使 `map_chain` 执行了另外两个。

```python
%%timeit
joke_chain.invoke({"topic": "bear"})
```

```output
610 毫秒 ± 64 毫秒每次循环（平均值 ± 7 次，1 次循环每次）
```

```python
%%timeit
poem_chain.invoke({"topic": "bear"})
```

```output
599 毫秒 ± 73.3 毫秒每次循环（平均值 ± 7 次，1 次循环每次）
```

```python
%%timeit
map_chain.invoke({"topic": "bear"})
```

```output
643 毫秒 ± 77.8 毫秒每次循环（平均值 ± 7 次，1 次循环每次）
```

## 下一步

您现在了解了一些使用 `RunnableParallel` 格式化和并行化链步骤的方法。

要了解更多信息，请参阅本部分关于可运行对象的其他操作指南。