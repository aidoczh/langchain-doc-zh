---

关键词: [LCEL]

---

# 如何创建动态（自构建）链

:::info 先决条件

本指南假定您熟悉以下内容：

- [LangChain 表达语言（LCEL）](/docs/concepts/#langchain-expression-language)

- [如何将任何函数转换为可运行形式](/docs/how_to/functions)

:::

有时我们希望根据链的输入在运行时构建链的部分（[路由](/docs/how_to/routing/)是最常见的例子）。我们可以利用 RunnableLambda 的一个非常有用的特性来创建这样的动态链，即如果一个 RunnableLambda 返回一个 Runnable，那么该 Runnable 本身将被调用。让我们看一个例子。

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="llm"
/>
```

```python
# | echo: false
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-3-sonnet-20240229")
```

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import Runnable, RunnablePassthrough, chain
contextualize_instructions = """将最新的用户问题转换为基于聊天历史的独立问题。不要回答问题，只返回问题本身（不包括描述性文本）。"""
contextualize_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_instructions),
        ("placeholder", "{chat_history}"),
        ("human", "{question}"),
    ]
)
contextualize_question = contextualize_prompt | llm | StrOutputParser()
qa_instructions = (
    """根据以下上下文回答用户问题：\n\n{context}。"""
)
qa_prompt = ChatPromptTemplate.from_messages(
    [("system", qa_instructions), ("human", "{question}")]
)
@chain
def contextualize_if_needed(input_: dict) -> Runnable:
    if input_.get("chat_history"):
        # 注意：这里返回的是另一个 Runnable，而不是实际输出。
        return contextualize_question
    else:
        return RunnablePassthrough()
@chain
def fake_retriever(input_: dict) -> str:
    return "2024年埃及的人口约为1.11亿"
full_chain = (
    RunnablePassthrough.assign(question=contextualize_if_needed).assign(
        context=fake_retriever
    )
    | qa_prompt
    | llm
    | StrOutputParser()
)
full_chain.invoke(
    {
        "question": "埃及情况如何",
        "chat_history": [
            ("human", "印度尼西亚的人口是多少"),
            ("ai", "大约2.76亿"),
        ],
    }
)
```

```output
"根据提供的上下文，2024年埃及的人口估计约为1.11亿。"
```

关键在于 `contextualize_if_needed` 返回的是另一个 Runnable，而不是实际输出。当执行完整链时，这个返回的 Runnable 本身会被运行。

通过查看跟踪，我们可以看到，由于我们传入了 chat_history，我们在完整链中执行了 contextualize_question 链：[链接](https://smith.langchain.com/public/9e0ae34c-4082-4f3f-beed-34a2a2f4c991/r)

请注意，返回的 Runnable 的流式处理、批处理等功能都得到了保留。

```python
for chunk in contextualize_if_needed.stream(
    {
        "question": "埃及情况如何",
        "chat_history": [
            ("human", "印度尼西亚的人口是多少"),
            ("ai", "大约2.76亿"),
        ],
    }
):
    print(chunk)
```

```output
埃及
的
人口
是
多少
？
```