# 如何使用多模态数据调用工具

在这里，我们将演示如何使用多模态数据（例如图像）调用工具。

一些多模态模型，例如可以对图像或音频进行推理的模型，也支持[工具调用](/docs/concepts/#functiontool-calling)功能。

要使用这些模型调用工具，只需按照[通常的方式](/docs/how_to/tool_calling)将工具绑定到模型中，并使用所需类型的内容块（例如包含图像数据的内容块）调用模型。

下面，我们将使用[OpenAI](/docs/integrations/platforms/openai)和[Anthropic](/docs/integrations/platforms/anthropic)来演示示例。我们将在所有情况下使用相同的图像和工具。首先选择一张图像，并构建一个期望输入为字符串"sunny"、"cloudy"或"rainy"的占位符工具。我们将要求模型描述图像中的天气。

```python
from typing import Literal
from langchain_core.tools import tool
image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
@tool
def weather_tool(weather: Literal["sunny", "cloudy", "rainy"]) -> None:
    """描述天气"""
    pass
```

## OpenAI

对于OpenAI，我们可以直接将图像URL作为类型为"image_url"的内容块输入：

```python
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-4o").bind_tools([weather_tool])
message = HumanMessage(
    content=[
        {"type": "text", "text": "describe the weather in this image"},
        {"type": "image_url", "image_url": {"url": image_url}},
    ],
)
response = model.invoke([message])
print(response.tool_calls)
```

```output
[{'name': 'weather_tool', 'args': {'weather': 'sunny'}, 'id': 'call_mRYL50MtHdeNuNIjSCm5UPmB'}]
```

请注意，我们在模型的响应中以LangChain的[标准格式](/docs/how_to/tool_calling)恢复了带有解析参数的工具调用。

## Anthropic

对于Anthropic，我们可以将base64编码的图像格式化为类型为"image"的内容块，如下所示：

```python
import base64
import httpx
from langchain_anthropic import ChatAnthropic
image_data = base64.b64encode(httpx.get(image_url).content).decode("utf-8")
model = ChatAnthropic(model="claude-3-sonnet-20240229").bind_tools([weather_tool])
message = HumanMessage(
    content=[
        {"type": "text", "text": "describe the weather in this image"},
        {
            "type": "image",
            "source": {
                "type": "base64",
                "media_type": "image/jpeg",
                "data": image_data,
            },
        },
    ],
)
response = model.invoke([message])
print(response.tool_calls)
```

```output
[{'name': 'weather_tool', 'args': {'weather': 'sunny'}, 'id': 'toolu_016m9KfknJqx5fVRYk4tkF6s'}]
```