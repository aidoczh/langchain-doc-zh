---

sidebar_label: OpenAI

---

# ChatOpenAI

这篇笔记介绍了如何开始使用 OpenAI 的聊天模型。

```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
```

```python
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
```

上述代码假定您已在环境变量中设置了 OpenAI API 密钥。如果您更愿意手动指定 API 密钥和/或组织 ID，请使用以下代码：

```python
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0, api_key="YOUR_API_KEY", openai_organization="YOUR_ORGANIZATION_ID")
```

如果不适用于您，可以删除 openai_organization 参数。

```python
messages = [
    ("system", "You are a helpful assistant that translates English to French."),
    ("human", "Translate this sentence from English to French. I love programming."),
]
llm.invoke(messages)
```

```output
AIMessage(content="J'adore programmer.", response_metadata={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 34, 'total_tokens': 40}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'stop', 'logprobs': None}, id='run-8591eae1-b42b-402b-a23a-dfdb0cd151bd-0')
```

## 链接

我们可以将我们的模型与提示模板链接起来，如下所示：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)
chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```

```output
AIMessage(content='Ich liebe Programmieren.', response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 26, 'total_tokens': 31}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'stop', 'logprobs': None}, id='run-94fa6741-c99b-4513-afce-c3f562631c79-0')
```

## 工具调用

OpenAI 有一个 [工具调用](https://platform.openai.com/docs/guides/function-calling) API，让您描述工具及其参数，并让模型返回一个 JSON 对象，其中包含要调用的工具以及该工具的输入。工具调用对于构建使用工具的链和代理非常有用，并且通常用于从模型中获取结构化输出。

### ChatOpenAI.bind_tools()

通过 `ChatOpenAI.bind_tools`，我们可以轻松地将 Pydantic 类、字典模式、LangChain 工具甚至函数作为工具传递给模型。在幕后，这些将被转换为一个 OpenAI 工具模式，看起来像这样：

```
{
    "name": "...",
    "description": "...",
    "parameters": {...}  # JSONSchema
}
```

并在每次模型调用中传递。

```python
from langchain_core.pydantic_v1 import BaseModel, Field
class GetWeather(BaseModel):
    """获取给定位置的当前天气"""
    location: str = Field(..., description="城市和州，例如旧金山，加利福尼亚州")
llm_with_tools = llm.bind_tools([GetWeather])
```

```python
ai_msg = llm_with_tools.invoke(
    "what is the weather like in San Francisco",
)
ai_msg
```

```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_H7fABDuzEau48T10Qn0Lsh0D', 'function': {'arguments': '{"location":"San Francisco"}', 'name': 'GetWeather'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 15, 'prompt_tokens': 70, 'total_tokens': 85}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b469135e-2718-446a-8164-eef37e672ba2-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco'}, 'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}])
```

### AIMessage.tool_calls

请注意，AIMessage 中有一个 `tool_calls` 属性。这包含了一个标准化的 ToolCall 格式，与模型提供者无关。

```python
ai_msg.tool_calls
```

```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco'},
  'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}]
```

要了解更多关于绑定工具和工具调用输出的信息，请查看 [tool calling](/docs/how_to/function_calling) 文档。

## 微调

您可以通过传入相应的 `modelName` 参数来调用微调的 OpenAI 模型。

通常采用 `ft:{OPENAI_MODEL_NAME}:{ORG_NAME}::{MODEL_ID}` 的形式。例如：

```python
fine_tuned_model = ChatOpenAI(
    temperature=0, model_name="ft:gpt-3.5-turbo-0613:langchain::7qTVM5AR"
)
fine_tuned_model(messages)
```

```output
AIMessage(content="J'adore la programmation.", additional_kwargs={}, example=False)
```