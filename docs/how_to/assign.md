# 如何向链的状态中添加值

:::info 先决条件

本指南假设您熟悉以下概念：

- [LangChain 表达式语言（LCEL）](/docs/concepts/#langchain-expression-language)

- [链接可运行项](/docs/how_to/sequence/)

- [并行调用可运行项](/docs/how_to/parallel/)

- [自定义函数](/docs/how_to/functions/)

- [数据传递](/docs/how_to/passthrough)

:::

[数据传递](/docs/how_to/passthrough)链的步骤之间的另一种方法是在给定的键下保持链状态的当前值不变，同时为其分配一个新值。[`RunnablePassthrough.assign()`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html#langchain_core.runnables.passthrough.RunnablePassthrough.assign) 静态方法接受一个输入值，并将传递给分配函数的额外参数添加到输入值中。

这在常见的[LangChain 表达式语言](/docs/concepts/#langchain-expression-language)模式中非常有用，该模式通过逐步创建一个字典来作为后续步骤的输入。

以下是一个示例：

```python
%pip install --upgrade --quiet langchain langchain-openai
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
runnable = RunnableParallel(
    extra=RunnablePassthrough.assign(mult=lambda x: x["num"] * 3),
    modified=lambda x: x["num"] + 1,
)
runnable.invoke({"num": 1})
```

```output
{'extra': {'num': 1, 'mult': 3}, 'modified': 2}
```

让我们分解一下这里发生了什么。

- 链的输入是 `{"num": 1}`。它传递给 `RunnableParallel`，该函数并行调用传递给它的可运行项。

- 调用了 `extra` 键的值。`RunnablePassthrough.assign()` 保留了输入字典中的原始键 (`{"num": 1}`)，并分配了一个名为 `mult` 的新键。值为 `lambda x: x["num"] * 3)`，即 `3`。因此，结果为 `{"num": 1, "mult": 3}`。

- `{"num": 1, "mult": 3}` 被返回给 `RunnableParallel` 调用，并设置为键 `extra` 的值。

- 同时，调用了 `modified` 键。结果为 `2`，因为 lambda 从其输入中提取了一个名为 `"num"` 的键，并加上了 `1`。

因此，结果为 `{'extra': {'num': 1, 'mult': 3}, 'modified': 2}`。

## 流式传递

这种方法的一个便利特性是它允许值在可用时立即通过。为了展示这一点，我们将使用 `RunnablePassthrough.assign()` 来立即返回检索链中的源文档：

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
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()
generation_chain = prompt | model | StrOutputParser()
retrieval_chain = {
    "context": retriever,
    "question": RunnablePassthrough(),
} | RunnablePassthrough.assign(output=generation_chain)
stream = retrieval_chain.stream("where did harrison work?")
for chunk in stream:
    print(chunk)
```

```output
{'question': 'where did harrison work?'}
{'context': [Document(page_content='harrison worked at kensho')]}
{'output': ''}
{'output': 'H'}
{'output': 'arrison'}
{'output': ' worked'}
{'output': ' at'}
{'output': ' Kens'}
{'output': 'ho'}
{'output': '.'}
{'output': ''}
```

我们可以看到，第一个块包含原始的 `"question"`，因为它是立即可用的。第二个块包含 `"context"`，因为检索器第二个完成。最后，`generation_chain` 的输出在可用时以块的形式流出。

## 下一步

现在您已经学会了如何通过链传递数据，以帮助格式化流经链的数据。

要了解更多信息，请参阅本节中有关可运行项的其他指南。