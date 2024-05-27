---

sidebar_position: 5

keywords: [RunnablePassthrough, LCEL]

---

# 如何将参数从一步传递到下一步

:::info 先决条件

本指南假定您熟悉以下概念：

- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)

- [链接可运行项](/docs/how_to/sequence/)

- [并行调用可运行项](/docs/how_to/parallel/)

- [自定义函数](/docs/how_to/functions/)

:::

在组合具有多个步骤的链时，有时您希望将数据从先前的步骤不经修改地传递到后续步骤以供使用。[`RunnablePassthrough`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html) 类允许您做到这一点，并且通常与 [RunnableParallel](/docs/how_to/parallel/) 一起使用，以将数据传递到构建的链中的后续步骤。

请参阅下面的示例：

```python
%pip install -qU langchain langchain-openai
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
runnable = RunnableParallel(
    passed=RunnablePassthrough(),
    modified=lambda x: x["num"] + 1,
)
runnable.invoke({"num": 1})
```

```output
{'passed': {'num': 1}, 'modified': 2}
```

如上所示，`passed` 键使用了 `RunnablePassthrough()`，因此它只是传递了 `{'num': 1}`。

我们还在映射中设置了第二个键 `modified`。这使用 lambda 来设置一个值，将 num 加 1，导致 `modified` 键的值为 `2`。

## 检索示例

在下面的示例中，我们看到了一个更实际的用例，其中我们在链中使用 `RunnablePassthrough` 以及 `RunnableParallel` 来正确格式化输入到提示：

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

这里，提示的输入预期是一个带有 "context" 和 "question" 键的映射。用户输入只是问题。因此，我们需要使用我们的检索器获取上下文，并将用户输入传递到 "question" 键下。`RunnablePassthrough` 允许我们将用户的问题传递给提示和模型。

## 下一步

现在您已经学会了如何通过链传递数据，以帮助格式化数据在链中流动。

要了解更多信息，请参阅本节中有关可运行项的其他指南。