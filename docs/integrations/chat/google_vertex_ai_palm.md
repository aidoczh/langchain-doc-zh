# ChatVertexAI

注意：这与 Google PaLM 集成是分开的。Google 选择通过 GCP 提供 PaLM 的企业版，并支持通过 GCP 提供的模型。

ChatVertexAI 公开了 Google Cloud 中所有基础模型：

- Gemini (`gemini-pro` 和 `gemini-pro-vision`)

- PaLM 2 for Text (`text-bison`)

- Codey for Code Generation (`codechat-bison`)

要查看完整和更新的可用模型列表，请访问 [VertexAI 文档](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/overview)。

默认情况下，Google Cloud [不使用](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development) 客户数据来训练其基础模型，作为 Google Cloud 的 AI/ML 隐私承诺的一部分。关于 Google 如何处理数据的更多详细信息也可以在 [Google 的客户数据处理附录 (CDPA)](https://cloud.google.com/terms/data-processing-addendum) 中找到。

要使用 `Google Cloud Vertex AI` PaLM，您必须安装 `langchain-google-vertexai` Python 包，并且要么：

- 针对您的环境配置了凭据 (gcloud、工作负载身份验证等...)

- 将服务帐号 JSON 文件的路径存储为 GOOGLE_APPLICATION_CREDENTIALS 环境变量

此代码库使用 `google.auth` 库，该库首先查找上述应用凭据变量，然后查找系统级别的身份验证。

更多信息，请参阅：

- https://cloud.google.com/docs/authentication/application-default-credentials#GAC

- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth

```python
%pip install --upgrade --quiet  langchain-google-vertexai
```
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_google_vertexai import ChatVertexAI
```
```python
system = "You are a helpful assistant who translate English to French"
human = "Translate this sentence from English to French. I love programming."
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
chat = ChatVertexAI()
chain = prompt | chat
chain.invoke({})
```
```output
AIMessage(content=" J'aime la programmation.")
```

目前 Gemini 不支持 SystemMessage，但可以将其添加到行中的第一个人类消息。如果需要这样的行为，只需将 `convert_system_message_to_human` 设置为 `True`：

```python
system = "You are a helpful assistant who translate English to French"
human = "Translate this sentence from English to French. I love programming."
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
chat = ChatVertexAI(model="gemini-pro", convert_system_message_to_human=True)
chain = prompt | chat
chain.invoke({})
```
```output
AIMessage(content="J'aime la programmation.")
```

如果我们想构建一个简单的链，以接受用户指定的参数：

```python
system = "You are a helpful assistant that translates {input_language} to {output_language}."
human = "{text}"
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
chat = ChatVertexAI()
chain = prompt | chat
chain.invoke(
    {
        "input_language": "English",
        "output_language": "Japanese",
        "text": "I love programming",
    }
)
```
```output
AIMessage(content=' プログラミングが大好きです')
```

## 代码生成聊天模型

您现在可以在 Vertex AI 中利用 Codey API 进行代码聊天。可用的模型有：

- `codechat-bison`: 用于代码辅助

```python
chat = ChatVertexAI(model="codechat-bison", max_tokens=1000, temperature=0.5)
message = chat.invoke("Write a Python function generating all prime numbers")
print(message.content)
```
```output
```python
def is_prime(n):
  """
  Check if a number is prime.
  Args:
    n: The number to check.
  Returns:
    True if n is prime, False otherwise.
  """
  # If n is 1, it is not prime.
  if n == 1:
    return False
  # Iterate over all numbers from 2 to the square root of n.
  for i in range(2, int(n ** 0.5) + 1):
    # If n is divisible by any number from 2 to its square root, it is not prime.
    if n % i == 0:
      return False
  # If n is divisible by no number from 2 to its square root, it is prime.
  return True
def find_prime_numbers(n):
  """
  Find all prime numbers up to a given number.
  Args:
    n: The upper bound for the prime numbers to find.
  Returns:
    A list of all prime numbers up to n.
  """
  # Create a list of all numbers from 2 to n.
  numbers = list(range(2, n + 1))
  # Iterate over the list of numbers and remove any that are not prime.
  for number in numbers:
    if not is_prime(number):
```
```markdown

## 完整生成信息

我们可以使用 `generate` 方法来获取额外的元数据，比如 [安全属性](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai#safety_attribute_confidence_scoring)，而不仅仅是聊天完成。

请注意，`generation_info` 将根据您是否使用 Gemini 模型而有所不同。

### Gemini 模型

`generation_info` 将包括：

- `is_blocked`：生成是否被阻止

- `safety_ratings`：安全评级的类别和概率标签

```python
from pprint import pprint
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import HarmBlockThreshold, HarmCategory
```
```python
human = "将这句话从英语翻译成法语。我喜欢编程。"
messages = [HumanMessage(content=human)]
chat = ChatVertexAI(
    model_name="gemini-pro",
    safety_settings={
        HarmCategory.HARM_CATEGORY_HATE_SPEECH: HarmBlockThreshold.BLOCK_LOW_AND_ABOVE
    },
)
result = chat.generate([messages])
pprint(result.generations[0][0].generation_info)
```
```output
{'citation_metadata': None,
 'is_blocked': False,
 'safety_ratings': [{'blocked': False,
                     'category': 'HARM_CATEGORY_HATE_SPEECH',
                     'probability_label': 'NEGLIGIBLE'},
                    {'blocked': False,
                     'category': 'HARM_CATEGORY_DANGEROUS_CONTENT',
                     'probability_label': 'NEGLIGIBLE'},
                    {'blocked': False,
                     'category': 'HARM_CATEGORY_HARASSMENT',
                     'probability_label': 'NEGLIGIBLE'},
                    {'blocked': False,
                     'category': 'HARM_CATEGORY_SEXUALLY_EXPLICIT',
                     'probability_label': 'NEGLIGIBLE'}],
 'usage_metadata': {'candidates_token_count': 6,
                    'prompt_token_count': 12,
                    'total_token_count': 18}}
```

### 非 Gemini 模型

`generation_info` 将包括：

- `is_blocked`：生成是否被阻止

- `safety_attributes`：将安全属性映射到其分数的字典

```python
chat = ChatVertexAI()  # 默认为 `chat-bison`
result = chat.generate([messages])
pprint(result.generations[0][0].generation_info)
```
```output
{'errors': (),
 'grounding_metadata': {'citations': [], 'search_queries': []},
 'is_blocked': False,
 'safety_attributes': [{'Derogatory': 0.1, 'Insult': 0.1, 'Sexual': 0.2}],
 'usage_metadata': {'candidates_billable_characters': 88.0,
                    'candidates_token_count': 24.0,
                    'prompt_billable_characters': 58.0,
                    'prompt_token_count': 12.0}}
```

## 工具调用（又名函数调用）与 Gemini

我们可以将工具定义传递给 Gemini 模型，以便在适当时候让模型调用这些工具。这不仅对于由 LLM 提供动力的工具使用有用，还可以更普遍地从模型中获取结构化输出。

通过 `ChatVertexAI.bind_tools()`，我们可以轻松地传递 Pydantic 类、字典模式、LangChain 工具，甚至函数作为工具传递给模型。在幕后，这些将被转换为 Gemini 工具模式，看起来像：

```python
{
    "name": "...",  # 工具名称
    "description": "...",  # 工具描述
    "parameters": {...}  # 工具输入模式为 JSONSchema
}
```
```python
from langchain.pydantic_v1 import BaseModel, Field
class GetWeather(BaseModel):
    """获取给定位置的当前天气"""
    location: str = Field(..., description="城市和州，例如旧金山，CA")
llm = ChatVertexAI(model="gemini-pro", temperature=0)
llm_with_tools = llm.bind_tools([GetWeather])
ai_msg = llm_with_tools.invoke(
    "旧金山的天气如何",
)
ai_msg
```
```output
AIMessage(content='', additional_kwargs={'function_call': {'name': 'GetWeather', 'arguments': '{"location": "San Francisco, CA"}'}}, response_metadata={'is_blocked': False, 'safety_ratings': [{'category': 'HARM_CATEGORY_HATE_SPEECH', 'probability_label': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_DANGEROUS_CONTENT', 'probability_label': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_HARASSMENT', 'probability_label': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_SEXUALLY_EXPLICIT', 'probability_label': 'NEGLIGIBLE', 'blocked': False}], 'citation_metadata': None, 'usage_metadata': {'prompt_token_count': 41, 'candidates_token_count': 7, 'total_token_count': 48}}, id='run-05e760dc-0682-4286-88e1-5b23df69b083-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, 'id': 'cd2499c4-4513-4059-bfff-5321b6e922d0'}])
```

工具调用可以通过 `AIMessage.tool_calls` 属性访问，其中以模型无关的格式提取出来：

```python
ai_msg.tool_calls
```
```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco, CA'},
  'id': 'cd2499c4-4513-4059-bfff-5321b6e922d0'}]
```

有关工具调用的完整指南，请参阅[这里](/docs/how_to/function_calling)。

## 结构化输出

许多应用程序需要结构化的模型输出。工具调用使得这一过程更加可靠和简单。[with_structured_outputs](https://api.python.langchain.com/en/latest/chat_models/langchain_google_vertexai.chat_models.ChatVertexAI.html) 构造函数提供了一个简单的接口，用于从模型中获取结构化输出。有关结构化输出的完整指南，请参阅[这里](/docs/how_to/structured_output)。

### ChatVertexAI.with_structured_outputs()

要从我们的 Gemini 模型中获取结构化输出，我们只需要指定一个所需的模式，可以是 Pydantic 类或 JSON 模式：

```python
class Person(BaseModel):
    """保存有关人员的信息。"""
    name: str = Field(..., description="人员姓名。")
    age: int = Field(..., description="人员年龄。")
structured_llm = llm.with_structured_output(Person)
structured_llm.invoke("Stefan 已经 13 岁了")
```
```output
Person(name='Stefan', age=13)
```

### [Legacy] 使用 `create_structured_runnable()`

获取结构化输出的传统方法是使用 `create_structured_runnable` 构造函数：

```python
from langchain_google_vertexai import create_structured_runnable
chain = create_structured_runnable(Person, llm)
chain.invoke("我的名字是 Erick，我今年 27 岁")
```

## 异步调用

我们可以通过 Runnables 的[异步接口](/docs/concepts#interface)进行异步调用。

```python
# 在笔记本中运行这些示例时：
import asyncio
import nest_asyncio
nest_asyncio.apply()
```
```python
system = (
    "您是一个有帮助的助手，可以将 {input_language} 翻译成 {output_language}。"
)
human = "{text}"
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
chat = ChatVertexAI(model="chat-bison", max_tokens=1000, temperature=0.5)
chain = prompt | chat
asyncio.run(
    chain.ainvoke(
        {
            "input_language": "英语",
            "output_language": "梵语",
            "text": "我喜欢编程",
        }
    )
)
```
```output
AIMessage(content=' अहं प्रोग्रामनं प्रेमामि')
```

## 流式调用

我们还可以通过 `stream` 方法流式输出：

```python
import sys
prompt = ChatPromptTemplate.from_messages(
    [("human", "列出世界上人口最多的五个国家")]
)
chat = ChatVertexAI()
chain = prompt | chat
for chunk in chain.stream({}):
    sys.stdout.write(chunk.content)
    sys.stdout.flush()
```
```output
世界上人口最多的五个国家是：
1. 中国（14 亿）
2. 印度（13 亿）
3. 美国（3.31 亿）
4. 印度尼西亚（2.73 亿）
5. 巴基斯坦（2.2 亿）
```