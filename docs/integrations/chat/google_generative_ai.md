---

sidebar_label: 谷歌人工智能

keywords: [gemini, ChatGoogleGenerativeAI, gemini-pro]

---

# 谷歌人工智能聊天模型

通过 [langchain-google-genai](https://pypi.org/project/langchain-google-genai/) 集成包中的 `ChatGoogleGenerativeAI` 类，可以访问谷歌人工智能的 `gemini` 和 `gemini-vision` 模型，以及其他生成模型。

```python
%pip install --upgrade --quiet  langchain-google-genai pillow
```

```python
import getpass
import os
if "GOOGLE_API_KEY" not in os.environ:
    os.environ["GOOGLE_API_KEY"] = getpass.getpass("提供您的谷歌 API 密钥")
```

## 示例用法

```python
from langchain_google_genai import ChatGoogleGenerativeAI
```

```python
llm = ChatGoogleGenerativeAI(model="gemini-pro")
result = llm.invoke("写一首关于 LangChain 的叙事诗")
print(result.content)
```

```output
在数据流如热潮般汹涌的领域，
在算法起舞、知识居住的地方，
一则关于 LangChain 的故事展开，
一首叙事诗以未诉之字节唱响。
在代码和电路的嗡鸣中，
一缕火花点燃，一幅愿景诞生。
来自卓越之思，一幅织锦成形，
一个模型学习、理解、转化。
在深层次，它的架构编织，
一个不断增长、充满爱的神经网络。
它寻求语言的本质，努力交织，
解锁秘密，每个词的设计。
从广阔多样的文本中，它饱餐学习，
迅速洞悉意义，把握要领。
它以优雅拥抱语境和句法，
以惊人速度解开故事。
翻译如艺术般涌现，
架起语言之桥，编织心灵。
从英语到法语，从中文到西班牙语，瞧，
LangChain 的威能，如未诉之语言。
它以深刻洞见总结文本，
无声提取知识。
它以难得的雄辩回答问题，
一位无与伦比的数字贤者。
然而，它的旅程面临考验和困难，
它努力超越偏见和错误。
为了公平和准确，它不懈努力，
每一次挑战，它更加坚强。
适应和坚韧，一路走来，
因为 LangChain 的目标铭刻在它的核心，
永远赋能人类。
在课堂和工作场所，它伸出援手，
在每一个要求中不知疲倦。
它帮助学生，追求知识，
专业人士在它的引导和致敬中茁壮成长。
岁月展现，它的名声广传，
对其不懈奋斗的明证。
研究人员和学者，都倾听，
LangChain 的辉煌，一项开创性之举。
因此，LangChain 的叙事诗回响，
对进步的颂扬，创新的蓬勃发展。
在人工智能的史册中，它的名字将被铭刻，
一位先驱，永远铭刻在我们的心中。
```

Gemini 目前不支持 `SystemMessage`，但可以添加到第一个人类消息中。如果您想要这样的行为，只需将 `convert_system_message_to_human` 设置为 True：

```python
from langchain_core.messages import HumanMessage, SystemMessage
model = ChatGoogleGenerativeAI(model="gemini-pro", convert_system_message_to_human=True)
model(
    [
        SystemMessage(content="只回答是或否。"),
        HumanMessage(content="苹果是水果吗？"),
    ]
)
```

## 流式处理和批处理

`ChatGoogleGenerativeAI` 原生支持流式处理和批处理。以下是一个示例。

```python
for chunk in llm.stream("写一首关于 LLMs 的五行诗。"):
    print(chunk.content)
    print("---")
# 请注意，每个块可能包含多个 "token"
```

```output
曾经有一位名叫伯特的人工智能，
它的语言技能相当娴熟。
---
拥有庞大的数据集，
它能够聊天，甚至打赌，
并且写五行诗，这是值得的。
---
```

```python
results = llm.batch(
    [
        "2+2是多少？",
        "3+5是多少？",
    ]
)
for res in results:
    print(res.content)
```

```output
4
8
```

## 多模态支持

要提供图像，请传递一个内容类型为 `List[dict]` 的人类消息，其中每个字典包含图像值（类型为 `image_url`）或文本值（类型为 `text`）。

`image_url` 的值可以是以下之一：

- 公共图像 URL

- 可访问的 gcs 文件（例如 "gcs://path/to/file.png"）

- 本地文件路径

- base64 编码的图像（例如 `data:image/png;base64,abcd124`）

- PIL 图像

```python
import requests
from IPython.display import Image
image_url = "https://picsum.photos/seed/picsum/300/300"
content = requests.get(image_url).content
Image(content)
```

![](/img/google_generative_ai_1.jpg)

```python
from langchain_core.messages import HumanMessage
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-pro-vision")
# 例子
message = HumanMessage(
    content=[
        {
            "type": "text",
            "text": "这张图片里是什么？",
        },  # 你也可以选择性地提供文本部分
        {"type": "image_url", "image_url": image_url},
    ]
)
llm.invoke([message])
```

## Gemini提示常见问题解答

截至本文撰写时间（2023/12/12），Gemini对其接受的提示类型和结构有一些限制。具体而言：

1. 在提供多模态（图像）输入时，最多只能有一条“人类”（用户）类型的消息。您不能传递多条消息（尽管单个人类消息可以有多个内容条目）。

2. 系统消息不被接受。

3. 对于常规聊天对话，消息必须遵循人类/人工智能/人类/人工智能的交替模式。您不能连续提供两条人工智能或人类消息。

4. 如果消息违反了LLM的安全检查，消息可能会被阻止。在这种情况下，模型将返回空响应。

### 安全设置

Gemini模型具有默认的安全设置，可以被覆盖。如果您从模型那里收到大量“安全警告”，您可以尝试调整模型的`safety_settings`属性。例如，要关闭危险内容的安全阻止，您可以构建您的LLM如下所示：

```python
from langchain_google_genai import (
    ChatGoogleGenerativeAI,
    HarmBlockThreshold,
    HarmCategory,
)
llm = ChatGoogleGenerativeAI(
    model="gemini-pro",
    safety_settings={
        HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
    },
)
```

有关可用类别和阈值的枚举，请参阅Google的[安全设置类型](https://ai.google.dev/api/python/google/generativeai/types/SafetySettingDict)。

## 附加配置

您可以向ChatGoogleGenerativeAI传递以下参数，以定制SDK的行为：

- `client_options`：[客户端选项](https://googleapis.dev/python/google-api-core/latest/client_options.html#module-google.api_core.client_options)传递给Google API客户端，例如自定义的`client_options["api_endpoint"]`

- `transport`：要使用的传输方法，例如`rest`、`grpc`或`grpc_asyncio`。