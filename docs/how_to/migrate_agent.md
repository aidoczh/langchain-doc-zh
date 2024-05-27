# 如何从传统的 LangChain 代理迁移到 LangGraph

本文将重点介绍如何从传统的 LangChain 代理迁移到 LangGraph 代理。

LangChain 代理（特别是 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor)）具有多个配置参数。

在本文中，我们将展示这些参数如何映射到 LangGraph 的 [react 代理执行器](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent)。

#### 先决条件

本指南使用 OpenAI 作为 LLM。安装依赖项以运行以下代码。

```python
%%capture --no-stderr
%pip install -U langgraph langchain langchain-openai
```

## 基本用法

对于基本的创建和使用工具调用 ReAct 风格代理，功能是相同的。首先，让我们定义一个模型和工具，然后我们将使用它们来创建一个代理。

```python
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-4o")
@tool
def magic_function(input: int) -> int:
    """对输入应用一个神奇的函数。"""
    return input + 2
tools = [magic_function]
query = "magic_function(3) 的值是多少？"
```

对于 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor)，我们定义了一个提示，其中包含代理的 scratchpad 占位符。代理可以如下所示调用：

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant"),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)
agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke({"input": query})
```

```output
{'input': 'magic_function(3) 的值是多少？',
 'output': 'magic_function(3) 的值是 5.'}
```

LangGraph 的 [react 代理执行器](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 管理着一个由消息列表定义的状态。它将继续处理列表，直到代理的输出中没有工具调用为止。为了启动它，我们输入一个消息列表。输出将包含整个图的状态--在本例中，是对话历史记录。

```python
from langgraph.prebuilt import create_react_agent
app = create_react_agent(model, tools)
messages = app.invoke({"messages": [("human", query)]})
{
    "input": query,
    "output": messages["messages"][-1].content,
}
```

```output
{'input': 'magic_function(3) 的值是多少？',
 'output': 'magic_function(3) 的值是 5.'}
```

```python
message_history = messages["messages"]
new_query = "请再说一遍？"
messages = app.invoke({"messages": message_history + [("human", new_query)]})
{
    "input": new_query,
    "output": messages["messages"][-1].content,
}
```

```output
{'input': '请再说一遍？',
 'output': '将 `magic_function` 应用于输入 3 的结果是 5.'}
```

## 提示模板

对于传统的 LangChain 代理，您必须传入一个提示模板。您可以使用它来控制代理。

对于 LangGraph 的 [react 代理执行器](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent)，默认情况下没有提示。您可以通过以下几种方式实现对代理的类似控制：

1. 将系统消息作为输入传入

2. 使用系统消息初始化代理

3. 使用一个函数初始化代理，该函数在传递给模型之前对消息进行转换。

让我们在下面看看所有这些方法。我们将传入自定义指令，让代理用西班牙语回复。

首先，使用 AgentExecutor：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Respond only in Spanish."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)
agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke({"input": query})
```

```output
{'input': 'magic_function(3) 的值是多少？',
 'output': 'magic_function(3) 的值是 5.'}
```

现在，让我们将自定义系统消息传递给 [react 代理执行器](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent)。这可以是字符串或 LangChain SystemMessage。

```python
from langchain_core.messages import SystemMessage
from langgraph.prebuilt import create_react_agent
system_message = "You are a helpful assistant. Respond only in Spanish."
# 这也可以是一个 SystemMessage 对象
# system_message = SystemMessage(content="You are a helpful assistant. Respond only in Spanish.")
app = create_react_agent(model, tools, messages_modifier=system_message)
messages = app.invoke({"messages": [("user", query)]})
```

我们还可以传入一个任意的函数。这个函数应该接收一个消息列表作为输入，并输出一个消息列表。

我们可以对消息进行任意的格式化。在这种情况下，让我们在消息列表的开头添加一个 SystemMessage。

```python
from langchain_core.messages import AnyMessage
from langgraph.prebuilt import create_react_agent
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一个乐于助人的助手。只用西班牙语回答。"),
        ("placeholder", "{messages}"),
    ]
)
def _modify_messages(messages: list[AnyMessage]):
    return prompt.invoke({"messages": messages}).to_messages() + [
        ("user", "回答后还要说“熊猫大乱斗！”")
    ]
app = create_react_agent(model, tools, messages_modifier=_modify_messages)
messages = app.invoke({"messages": [("human", query)]})
print(
    {
        "input": query,
        "output": messages["messages"][-1].content,
    }
)
```

```output
{'input': 'what is the value of magic_function(3)?', 'output': 'El valor de magic_function(3) es 5. ¡Pandamonium!'}
```

## 记忆

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter)，您可以添加聊天 [Memory](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.memory)，以便进行多轮对话。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-4o")
memory = ChatMessageHistory(session_id="test-session")
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一个乐于助人的助手。"),
        # 首先放入历史记录
        ("placeholder", "{chat_history}"),
        # 然后是新的输入
        ("human", "{input}"),
        # 最后是草稿
        ("placeholder", "{agent_scratchpad}"),
    ]
)
@tool
def magic_function(input: int) -> int:
    """对输入应用一个神奇的函数。"""
    return input + 2
tools = [magic_function]
agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    # 这是必需的，因为在大多数实际场景中，需要一个会话 ID
    # 在这里并没有真正使用，因为我们使用了一个简单的内存中的 ChatMessageHistory
    lambda session_id: memory,
    input_messages_key="input",
    history_messages_key="chat_history",
)
config = {"configurable": {"session_id": "test-session"}}
print(
    agent_with_chat_history.invoke(
        {"input": "Hi, I'm polly! What's the output of magic_function of 3?"}, config
    )["output"]
)
print("---")
print(agent_with_chat_history.invoke({"input": "Remember my name?"}, config)["output"])
print("---")
print(
    agent_with_chat_history.invoke({"input": "what was that output again?"}, config)[
        "output"
    ]
)
```

```output
Hi Polly! The output of the magic function for the input 3 is 5.
---
Yes, I remember your name, Polly! How can I assist you further?
---
The output of the magic function for the input 3 is 5.
```

#### 在 LangGraph 中

记忆只是 [持久化](https://langchain-ai.github.io/langgraph/how-tos/persistence/)，也就是 [检查点](https://langchain-ai.github.io/langgraph/reference/checkpoints/)。

将一个 `checkpointer` 添加到代理中，您就可以免费获得聊天记忆。

```python
from langchain_core.messages import SystemMessage
from langgraph.checkpoint import MemorySaver  # 一个内存中的检查点
from langgraph.prebuilt import create_react_agent
system_message = "你是一个乐于助人的助手。"
# 这也可以是一个 SystemMessage 对象
# system_message = SystemMessage(content="你是一个乐于助人的助手。只用西班牙语回答。")
memory = MemorySaver()
app = create_react_agent(
    model, tools, messages_modifier=system_message, checkpointer=memory
)
config = {"configurable": {"thread_id": "test-thread"}}
print(
    app.invoke(
        {
            "messages": [
                ("user", "Hi, I'm polly! What's the output of magic_function of 3?")
            ]
        },
        config,
    )["messages"][-1].content
)
print("---")
print(
    app.invoke({"messages": [("user", "Remember my name?")]}, config)["messages"][
        -1
    ].content
)
print("---")
print(
    app.invoke({"messages": [("user", "what was that output again?")]}, config)[
        "messages"
    ][-1].content
)
```

嗨，Polly！对于输入 3，magic_function 的输出是 5。

---

是的，你的名字是 Polly！

---

对于输入 3，magic_function 的输出是 5。

## 通过步骤迭代

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter)，你可以使用 [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream)（或异步的 `astream`）方法或 [iter](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter) 方法迭代步骤。LangGraph 支持使用 [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) 进行逐步迭代。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)
@tool
def magic_function(input: int) -> int:
    """对输入应用一个神奇的函数。"""
    return input + 2
tools = [magic_function]
agent = create_tool_calling_agent(model, tools, prompt=prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)
for step in agent_executor.stream({"input": query}):
    print(step)
```

```output
{'actions': [ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls'}, id='run-c68fd76f-a3c3-4c3c-bfd7-748c171ed4b8', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'index': 0}])], tool_call_id='call_q9MgGFjqJbV2xSUX93WqxmOt')], 'messages': [AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls'}, id='run-c68fd76f-a3c3-4c3c-bfd7-748c171ed4b8', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'index': 0}])]}
{'steps': [AgentStep(action=ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls'}, id='run-c68fd76f-a3c3-4c3c-bfd7-748c171ed4b8', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_q9MgGFjqJbV2xSUX93WqxmOt', 'index': 0}])], tool_call_id='call_q9MgGFjqJbV2xSUX93WqxmOt'), observation=5)], 'messages': [FunctionMessage(content='5', name='magic_function')]}
{'output': 'The value of `magic_function(3)` is 5.', 'messages': [AIMessage(content='The value of `magic_function(3)` is 5.')]}
```

#### 在 LangGraph 中

在 LangGraph 中，使用 [stream](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.graph.CompiledGraph.stream) 或异步的 `astream` 方法来处理事务。

```python
from langchain_core.messages import AnyMessage
from langgraph.prebuilt import create_react_agent
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("placeholder", "{messages}"),
    ]
)
def _modify_messages(messages: list[AnyMessage]):
    return prompt.invoke({"messages": messages}).to_messages()
app = create_react_agent(model, tools, messages_modifier=_modify_messages)
for step in app.stream({"messages": [("human", query)]}, stream_mode="updates"):
    print(step)
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_yTjXXibj76tyFyPRa1soLo0S', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 70, 'total_tokens': 84}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b275f314-c42e-4e77-9dec-5c23f7dbd53b-0', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_yTjXXibj76tyFyPRa1soLo0S'}])]}}
{'tools': {'messages': [ToolMessage(content='5', name='magic_function', id='41c5f227-528d-4483-a313-b03b23b1d327', tool_call_id='call_yTjXXibj76tyFyPRa1soLo0S')]}}
{'agent': {'messages': [AIMessage(content='The value of `magic_function(3)` is 5.', response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 93, 'total_tokens': 107}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'stop', 'logprobs': None}, id='run-0ef12b6e-415d-4758-9b62-5e5e1b350072-0')]}}
```

## `return_intermediate_steps`

在 AgentExecutor 上设置此参数允许用户访问 intermediate_steps，它将代理的动作（例如工具调用）与其结果进行配对。

```python
agent_executor = AgentExecutor(agent=agent, tools=tools, return_intermediate_steps=True)
result = agent_executor.invoke({"input": query})
print(result["intermediate_steps"])
```

```output
[(ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_ABI4hftfEdnVgKyfF6OzZbca', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls'}, id='run-837e794f-cfd8-40e0-8abc-4d98ced11b75', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_ABI4hftfEdnVgKyfF6OzZbca'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_ABI4hftfEdnVgKyfF6OzZbca', 'index': 0}])], tool_call_id='call_ABI4hftfEdnVgKyfF6OzZbca'), 5)]
```

默认情况下，LangGraph 应用中的 [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 会将所有消息附加到中央状态。因此，只需查看完整状态即可轻松查看任何中间步骤。

```python
from langgraph.prebuilt import create_react_agent
app = create_react_agent(model, tools=tools)
messages = app.invoke({"messages": [("human", query)]})
messages
```

```output
{'messages': [HumanMessage(content='what is the value of magic_function(3)?', id='0f63e437-c4d8-4da9-b6f5-b293ebfe4a64'),
  AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_S96v28LlI6hNkQrNnIio0JPh', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 64, 'total_tokens': 78}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-ffef7898-14b1-4537-ad90-7c000a8a5d25-0', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_S96v28LlI6hNkQrNnIio0JPh'}]),
  ToolMessage(content='5', name='magic_function', id='fbd9df4e-1dda-4d3e-9044-b001f7875476', tool_call_id='call_S96v28LlI6hNkQrNnIio0JPh'),
  AIMessage(content='The value of `magic_function(3)` is 5.', response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 87, 'total_tokens': 101}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'stop', 'logprobs': None}, id='run-e5d94c54-d9f4-45cd-be8e-a9101a8d88d6-0')]}
```

## `max_iterations`

`AgentExecutor` 实现了 `max_iterations` 参数，而在 LangGraph 中通过 `recursion_limit` 进行控制。

请注意，在 AgentExecutor 中，"iteration" 包括一次完整的工具调用和执行。在 LangGraph 中，每个步骤都会增加递归限制，因此我们需要乘以二（并加一）以获得相同的结果。

如果达到递归限制，LangGraph 会引发特定的异常类型，我们可以捕获并类似地处理，就像 AgentExecutor 一样。

```python
@tool
def magic_function(input: str) -> str:
    """对输入应用魔术函数。"""
    return "Sorry, there was an error. Please try again."
tools = [magic_function]
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Respond only in Spanish."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)
agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=3,
)
agent_executor.invoke({"input": query})
```

```output
> Entering new AgentExecutor chain...
Invoking: `magic_function` with `{'input': '3'}`
Sorry, there was an error. Please try again.
Invoking: `magic_function` with `{'input': '3'}`
responded: Parece que hubo un error al intentar obtener el valor de `magic_function(3)`. Permíteme intentarlo de nuevo.
Sorry, there was an error. Please try again.Aún no puedo obtener el valor de `magic_function(3)`. ¿Hay algo más en lo que pueda ayudarte?
> Finished chain.
```

```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'Aún no puedo obtener el valor de `magic_function(3)`. ¿Hay algo más en lo que pueda ayudarte?'}
```

```python
from langgraph.errors import GraphRecursionError
from langgraph.prebuilt import create_react_agent
RECURSION_LIMIT = 2 * 3 + 1
app = create_react_agent(model, tools=tools)
try:
    for chunk in app.stream(
        {"messages": [("human", query)]},
        {"recursion_limit": RECURSION_LIMIT},
```

```python
import asyncio
from langgraph.prebuilt import create_react_agent
app = create_react_agent(model, tools=tools)
async def stream(app, inputs):
    async for chunk in app.astream({"messages": [("human", query)]}):
        print(chunk)
        print("------")
loop = asyncio.get_event_loop()
loop.run_until_complete(stream(app, {"input": query}))
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_HaQkeCwD5QskzJzFixCBacZ4', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 64, 'total_tokens': 78}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-596c9200-771f-436d-8576-72fcb81620f1-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_HaQkeCwD5QskzJzFixCBacZ4'}])]}}
------
{'input': 'what is the value of magic_function(3)?', 'output': 'Agent stopped due to max iterations.'}
```

```python
agent_executor = AgentExecutor(
    agent=agent, tools=tools, trim_intermediate_steps=1
)
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_4agJXUHtmHrOOMogjF6ZuzAv', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 64, 'total_tokens': 78}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_729ea513f7', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-a1c77db7-405f-43d9-8d57-751f2ca1a58c-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_4agJXUHtmHrOOMogjF6ZuzAv'}])]}}
```

在 LangChain 中，您可以使用 [trim_intermediate_steps](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.trim_intermediate_steps) 来剪切长时间运行的代理的中间步骤。这个参数可以是一个整数（表示代理应保留最后 N 步），也可以是一个自定义函数。

例如，我们可以将值设为 1，这样代理就只会看到最近的中间步骤。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)
magic_step_num = 1
@tool
def magic_function(input: int) -> int:
    """对输入应用一个神奇的函数。"""
    global magic_step_num
    print(f"调用次数：{magic_step_num}")
    magic_step_num += 1
    return input + magic_step_num
tools = [magic_function]
agent = create_tool_calling_agent(model, tools, prompt=prompt)
def trim_steps(steps: list):
    # 让代理人失忆
    return []
agent_executor = AgentExecutor(
    agent=agent, tools=tools, trim_intermediate_steps=trim_steps
)
query = "连续调用神奇函数4次，初始值为3。你不能一次调用多次。"
for step in agent_executor.stream({"input": query}):
    pass
```

```output
调用次数：1
调用次数：2
调用次数：3
调用次数：4
调用次数：5
调用次数：6
调用次数：7
调用次数：8
调用次数：9
调用次数：10
调用次数：11
调用次数：12
调用次数：13
调用次数：14
```

```output
因触发停止条件而提前停止代理人
```

#### 在 LangGraph

我们可以像之前一样使用 [`messages_modifier`](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) ，在传入 [提示模板](#prompt-templates) 时使用。

```python
from langchain_core.messages import AnyMessage
from langgraph.errors import GraphRecursionError
from langgraph.prebuilt import create_react_agent
magic_step_num = 1
@tool
def magic_function(input: int) -> int:
    """对输入应用一个神奇的函数。"""
    global magic_step_num
    print(f"调用次数：{magic_step_num}")
    magic_step_num += 1
    return input + magic_step_num
tools = [magic_function]
def _modify_messages(messages: list[AnyMessage]):
    # 让代理人失忆，只保留原始用户查询
    return [("system", "You are a helpful assistant"), messages[0]]
app = create_react_agent(model, tools, messages_modifier=_modify_messages)
try:
    for step in app.stream({"messages": [("human", query)]}, stream_mode="updates"):
        pass
except GraphRecursionError as e:
    print("因触发停止条件而提前停止代理人")
```

```output
调用次数：1
调用次数：2
调用次数：3
调用次数：4
调用次数：5
调用次数：6
调用次数：7
调用次数：8
调用次数：9
调用次数：10
调用次数：11
调用次数：12
因触发停止条件而提前停止代理人
```