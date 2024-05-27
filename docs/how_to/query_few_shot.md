---

sidebar_position: 2

---

# 如何向提示中添加示例以进行查询分析

随着我们的查询分析变得更加复杂，LLM可能会在某些情况下难以理解应该如何准确地做出响应。为了在这里提高性能，我们可以向提示中添加示例来指导LLM。

让我们看看如何为我们在[快速入门](/docs/tutorials/query_analysis)中构建的LangChain YouTube视频查询分析器添加示例。

## 设置

#### 安装依赖

```python
# %pip install -qU langchain-core langchain-openai
```

#### 设置环境变量

在这个示例中，我们将使用OpenAI：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 可选，取消注释以使用LangSmith跟踪运行。在这里注册：https://smith.langchain.com.
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 查询模式

我们将定义一个查询模式，希望我们的模型输出该模式。为了使我们的查询分析更加有趣，我们将添加一个`sub_queries`字段，其中包含从顶层问题派生的更窄问题。

```python
from typing import List, Optional
from langchain_core.pydantic_v1 import BaseModel, Field
sub_queries_description = """\
如果原始问题包含多个不同的子问题，
或者有更通用的问题对于回答原始问题很有帮助，
请列出所有相关的子问题。
确保这个列表是全面的，覆盖原始问题的所有部分。
如果子问题中存在冗余，也没关系。
确保子问题尽可能专注。"""
class Search(BaseModel):
    """搜索关于软件库教程视频的数据库。"""
    query: str = Field(
        ...,
        description="应用于视频转录的主要相似性搜索查询。",
    )
    sub_queries: List[str] = Field(
        default_factory=list, description=sub_queries_description
    )
    publish_year: Optional[int] = Field(None, description="视频发布年份")
```

## 查询生成

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
system = """您是将用户问题转换为数据库查询的专家。
您可以访问一个关于构建LLM应用程序的软件库教程视频数据库。
给定一个问题，返回一组优化以检索最相关结果的数据库查询。
如果有您不熟悉的缩写或词汇，请不要尝试重新表达它们。"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        MessagesPlaceholder("examples", optional=True),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```

让我们尝试在提示中没有任何示例的情况下使用我们的查询分析器：

```python
query_analyzer.invoke(
    "web voyager和reflection agents之间有什么区别？它们都使用langgraph吗？"
)
```

```output
Search(query='web voyager vs reflection agents', sub_queries=['difference between web voyager and reflection agents', 'do web voyager and reflection agents use langgraph'], publish_year=None)
```

## 添加示例和调整提示

这个效果相当不错，但我们可能希望它进一步分解问题，以区分关于Web Voyager和Reflection Agents的查询。

为了调整我们的查询生成结果，我们可以向提示中添加一些输入问题和标准输出查询的示例。

```python
examples = []
```

```python
question = "什么是chat langchain，它是一个langchain模板吗？"
query = Search(
    query="What is chat langchain and is it a langchain template?",
    sub_queries=["What is chat langchain", "What is a langchain template"],
)
examples.append({"input": question, "tool_calls": [query]})
```

```python
question = "如何构建多代理系统并从中流式传输中间步骤"
query = Search(
    query="How to build multi-agent system and stream intermediate steps from it",
    sub_queries=[
        "How to build multi-agent system",
        "How to stream intermediate steps from multi-agent system",
        "How to stream intermediate steps",
    ],
)
examples.append({"input": question, "tool_calls": [query]})
```

```python
question = "LangChain agents和LangGraph有什么区别？"
query = Search(
    query="What's the difference between LangChain agents and LangGraph? How do you deploy them?",
    sub_queries=[
        "What are LangChain agents",
        "What is LangGraph",
        "How do you deploy LangChain agents",
        "How do you deploy LangGraph",
    ],
)
examples.append({"input": question, "tool_calls": [query]})
```

现在我们需要更新提示模板和链条，以便在每个提示中包含示例。由于我们使用的是OpenAI的函数调用，我们需要进行一些额外的结构化工作，以将示例输入和输出发送到模型。我们将创建一个`tool_example_to_messages`辅助函数来处理这个问题：

```python
import uuid
from typing import Dict
from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)
def tool_example_to_messages(example: Dict) -> List[BaseMessage]:
    messages: List[BaseMessage] = [HumanMessage(content=example["input"])]
    openai_tool_calls = []
    for tool_call in example["tool_calls"]:
        openai_tool_calls.append(
            {
                "id": str(uuid.uuid4()),
                "type": "function",
                "function": {
                    "name": tool_call.__class__.__name__,
                    "arguments": tool_call.json(),
                },
            }
        )
    messages.append(
        AIMessage(content="", additional_kwargs={"tool_calls": openai_tool_calls})
    )
    tool_outputs = example.get("tool_outputs") or [
        "You have correctly called this tool."
    ] * len(openai_tool_calls)
    for output, tool_call in zip(tool_outputs, openai_tool_calls):
        messages.append(ToolMessage(content=output, tool_call_id=tool_call["id"]))
    return messages
example_msgs = [msg for ex in examples for msg in tool_example_to_messages(ex)]
```

```python
from langchain_core.prompts import MessagesPlaceholder
query_analyzer_with_examples = (
    {"question": RunnablePassthrough()}
    | prompt.partial(examples=example_msgs)
    | structured_llm
)
```

```python
query_analyzer_with_examples.invoke(
    "what's the difference between web voyager and reflection agents? do both use langgraph?"
)
```

```output
Search(query='Difference between web voyager and reflection agents, do they both use LangGraph?', sub_queries=['What is Web Voyager', 'What are Reflection agents', 'Do Web Voyager and Reflection agents use LangGraph'], publish_year=None)
```

通过我们的示例，我们得到了一个稍微更详细的搜索查询。通过进一步的提示工程和调整示例，我们可以进一步改进查询生成。

您可以看到示例作为消息传递给模型的[LangSmith跟踪](https://smith.langchain.com/public/aeaaafce-d2b1-4943-9a61-bc954e8fc6f2/r)。