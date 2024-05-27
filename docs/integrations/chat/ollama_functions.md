---

sidebar_label: Ollama Functions

---

# OllamaFunctions

这篇笔记展示了如何使用一个实验性封装的 Ollama，使其具有与 OpenAI Functions 相同的 API。

需要注意的是，更强大和功能更全面的模型在处理复杂模式和/或多个函数时会表现更好。下面的示例使用了 llama3 和 phi3 模型。

有关支持的模型和模型变体的完整列表，请参阅 [Ollama 模型库](https://ollama.ai/library)。

## 设置

请按照 [这些说明](https://github.com/jmorganca/ollama) 来设置和运行本地的 Ollama 实例。

## 用法

您可以以类似于初始化标准 ChatOllama 实例的方式来初始化 OllamaFunctions：

```python
from langchain_experimental.llms.ollama_functions import OllamaFunctions
model = OllamaFunctions(model="llama3", format="json")
```

然后，您可以将使用 JSON Schema 参数和 `function_call` 参数定义的函数绑定到模型，以强制模型调用给定的函数：

```python
model = model.bind_tools(
    tools=[
        {
            "name": "get_current_weather",
            "description": "获取给定位置的当前天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市和州，例如 San Francisco, CA",
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                    },
                },
                "required": ["location"],
            },
        }
    ],
    function_call={"name": "get_current_weather"},
)
```

使用该模型调用函数将会产生与提供的模式匹配的 JSON 输出：

```python
from langchain_core.messages import HumanMessage
model.invoke("波士顿的天气如何？")
```

```output
AIMessage(content='', additional_kwargs={'function_call': {'name': 'get_current_weather', 'arguments': '{"location": "Boston, MA"}'}}, id='run-1791f9fe-95ad-4ca4-bdf7-9f73eab31e6f-0')
```

## 结构化输出

使用 `with_structured_output()` 函数进行函数调用的一个有用功能是以结构化格式从给定输入中提取属性：

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
# 结构化响应的模式
class Person(BaseModel):
    name: str = Field(description="人的姓名", required=True)
    height: float = Field(description="人的身高", required=True)
    hair_color: str = Field(description="人的头发颜色")
# 提示模板
prompt = PromptTemplate.from_template(
    """Alex 身高 5 英尺。 
Claudia 比 Alex 高 1 英尺，并且比他跳得更高。 
Claudia 是金发，Alex 是黑发。
Human: {question}
AI: """
)
# 链
llm = OllamaFunctions(model="phi3", format="json", temperature=0)
structured_llm = llm.with_structured_output(Person)
chain = prompt | structured_llm
```

### 提取关于 Alex 的数据

```python
alex = chain.invoke("描述一下 Alex")
alex
```

```output
Person(name='Alex', height=5.0, hair_color='黑发')
```

### 提取关于 Claudia 的数据

```python
claudia = chain.invoke("描述一下 Claudia")
claudia
```

```output
Person(name='Claudia', height=6.0, hair_color='金发')
```