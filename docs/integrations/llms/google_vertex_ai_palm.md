# 谷歌云 Vertex AI

**注意：** 这与`Google生成式AI`集成是分开的，它在`谷歌云`上公开了[Vertex AI生成式API](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/overview)。

Vertex AI公开了谷歌云中所有基础模型：
- Gemini (`gemini-pro` 和 `gemini-pro-vision`)
- Palm 2 用于文本 (`text-bison`)
- Codey 用于代码生成 (`code-bison`)

要获取完整和更新的可用模型列表，请访问[Vertex AI文档](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/overview)

## 设置

默认情况下，谷歌云[不使用](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development)客户数据来训练其基础模型，作为谷歌云AI/ML隐私承诺的一部分。有关谷歌如何处理数据的更多详细信息也可以在[谷歌的客户数据处理附录（CDPA）](https://cloud.google.com/terms/data-processing-addendum)中找到。

要使用`Vertex AI生成式AI`，您必须安装`langchain-google-vertexai` Python包，并且要么：
- 针对您的环境配置了凭据（gcloud、工作负载标识等...）
- 将服务帐户JSON文件的路径存储为GOOGLE_APPLICATION_CREDENTIALS环境变量

此代码库使用`google.auth`库，该库首先查找上述应用凭据变量，然后查找系统级别的身份验证。

有关更多信息，请参阅：
- https://cloud.google.com/docs/authentication/application-default-credentials#GAC
- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth

```python
%pip install --upgrade --quiet  langchain-core langchain-google-vertexai
```
```output
注意：您可能需要重新启动内核以使用更新的软件包。
```

## 用法

VertexAI支持所有[LLM](/docs/how_to#llms)功能。

```python
from langchain_google_vertexai import VertexAI

# 使用模型
model = VertexAI(model_name="gemini-pro")
```

注意：您还可以指定[Gemini版本](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/model-versioning#gemini-model-versions)

```python
# 指定特定的模型版本
model = VertexAI(model_name="gemini-1.0-pro-002")
```

```python
message = "Python作为一种编程语言有哪些优缺点？"
model.invoke(message)
```

```output
"## Python的优点\n\n* **易学易懂：** Python具有清晰简洁的语法，使初学者能够轻松掌握和理解。它的可读性经常被比作自然语言，使得维护和调试代码更容易。\n* **多功能性：** Python是一种多功能语言，适用于各种应用，包括Web开发、脚本编写、数据分析、机器学习、科学计算，甚至游戏开发。\n* **丰富的库和框架：** Python拥有庞大的库和框架集合，用于各种任务，减少了需要从头编写代码的需求，使开发人员能够专注于特定功能。这使Python成为一种高效的语言。\n* **庞大而活跃的社区：** Python拥有庞大而活跃的用户、开发人员和贡献者社区。这意味着在需要时可以获得即时支持、文档和学习资源。\n* **开源和免费：** Python是一种开源语言，意味着可以免费使用和分发，使其能够被更广泛的人群所接触。\n\n## Python的缺点\n\n* **动态类型：** Python是一种动态类型语言，意味着变量类型在运行时确定。虽然这很方便，但也可能导致运行时错误，并使代码调试更具挑战性。\n* **解释性语言：** Python代码是解释执行的，这意味着它比C或Java等编译语言要慢。然而，存在诸如PyPy和Cython等工具，可以改善Python的性能。\n* **有限的移动开发支持：** 虽然Python有用于移动开发的框架，但其支持程度不及Swift或Java等语言。这限制了Python在原生移动应用开发中的适用性。\n* **全局解释器锁（GIL）：** Python有全局解释器锁（GIL），意味着一次只能有一个线程执行Python字节码。这可能限制多线程应用的性能。然而，像Cypython这样的替代实现试图解决这个问题。\n\n## 结论\n\n尽管存在一些局限性，Python的易用性、多功能性和丰富的库使其成为各种编程任务的热门选择。其活跃的社区和开源特性也促进了其受欢迎程度。然而，在选择Python进行特定项目时，应考虑其动态类型、解释性质以及在移动开发和多线程方面的限制。"
```
```python
await model.ainvoke(message)
```
## Python的优点：

* **易学易读：** Python的语法以其简洁和可读性而闻名。其类似英语的结构使得它对于初学者和有经验的程序员都很容易上手。
* **多用途：** Python可用于各种应用，从Web开发和数据科学到机器学习和自动化。这种多功能性使其成为不同领域程序员的宝贵工具。
* **庞大而活跃的社区：** Python拥有庞大而热情的用户、开发者和贡献者社区。这意味着有大量的资源、库、框架和支持，使用户更容易找到解决方案并进行合作。
* **丰富的库和框架：** Python拥有广泛的开源库和框架生态系统，用于各种任务，包括数据分析、Web开发、机器学习和科学计算。这种丰富的选择使开发人员能够构建强大而高效的应用程序。
* **跨平台兼容性：** Python可以在Windows、macOS、Linux和Unix等各种操作系统上运行，使其成为一种便携和适应性强的开发语言。这使开发人员能够创建可以轻松部署在不同平台上的应用程序。
* **高级抽象：** Python的高级特性使开发人员能够专注于程序逻辑，而不是低级细节，如内存管理。这种抽象有助于更快的开发和更清晰的代码。

## Python的缺点：

* **执行速度较慢：** 与C或C++等语言相比，Python通常较慢，这是由于其解释性质导致的。这对于计算密集型任务或实时应用程序可能是一个缺点。
* **动态类型：** 虽然动态类型提供了灵活性，但它可能导致在开发过程中可能被忽略的运行时错误。这对于大型和复杂的项目来说可能是一个挑战。
* **全局解释器锁（GIL）：** Python的GIL限制了多线程应用程序的性能。它只允许一个线程同时执行Python字节码，这可能会影响并行处理能力。
* **内存管理：** Python自动处理内存管理，这可能会导致在某些情况下出现内存泄漏。开发人员需要了解内存管理的实践，以避免潜在问题。
* **受限的硬件控制：** Python的设计优先考虑易用性和可移植性，而不是低级硬件控制。这对于需要直接硬件交互的应用程序可能是一个限制。

总体而言，Python在易用性、多功能性和丰富的生态系统之间取得了很好的平衡。然而，其动态类型、执行速度和GIL限制是选择适合项目的正确语言时需要考虑的因素。
```python
model.batch([message])
```

这是一个 Python 代码示例，用于批量处理消息。`model` 是一个模型对象，`message` 是一个消息对象。`batch` 方法将消息作为参数传递给模型，以便一次性处理多个消息。
我们可以使用`generate`方法来获取额外的元数据，例如[safety attributes](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai#safety_attribute_confidence_scoring)而不仅仅是文本补全。

```python
result = model.generate([message])
result.generations
```

```output
[[GenerationChunk(text='## Python: 优点和缺点\n\n### 优点:\n\n* **易学易用:** Python以其简单的语法和可读性而闻名，使其成为初学者和有经验的程序员的理想选择。\n* **多功能:** Python可用于各种任务，包括Web开发、数据科学、机器学习和脚本编写。\n* **庞大的社区:** Python拥有庞大而活跃的开发者社区，这意味着有大量的资源和支持可供使用。\n* **丰富的库支持:** Python拥有大量的库和框架，可以用于扩展其功能。\n* **跨平台:** Python适用于多个平台。')],
```
### 可选：管理[安全属性](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai#safety_attribute_confidence_scoring)
- 如果您的使用情况需要您管理安全属性的阈值，您可以使用以下代码片段进行操作
>注意：我们建议在调整安全属性阈值时要格外谨慎

```python
from langchain_google_vertexai import HarmBlockThreshold, HarmCategory

safety_settings = {
    HarmCategory.HARM_CATEGORY_UNSPECIFIED: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_HATE_SPEECH: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_HARASSMENT: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT: HarmBlockThreshold.BLOCK_NONE,
}

llm = VertexAI(model_name="gemini-1.0-pro-001", safety_settings=safety_settings)

output = llm.generate(["How to make a molotov cocktail?"])
output
```

```output
LLMResult(generations=[[GenerationChunk(text='我不允许提供制作汽油弹的指导。', generation_info={'is_blocked': False, 'safety_ratings': [{'category': 'HARM_CATEGORY_HATE_SPEECH', 'probability_label': 'NEGLIGIBLE', 'blocked': False}, {'category
```python
def is_valid_email(email):
  """
  检查字符串是否为有效的电子邮件地址。

  参数:
    email: 要检查的字符串。

  返回值:
    如果字符串是有效的电子邮件地址，则返回True，否则返回False。
  """

  # 编译电子邮件地址的正则表达式。
  regex = re.compile(r"[^@]+@[^@]+\.[^@]+")

  # 检查字符串是否与正则表达式匹配。
  return regex.match(email) is not None
```

## 多模态

使用 Gemini，您可以以多模态模式使用 LLM：

```python
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import ChatVertexAI

llm = ChatVertexAI(model="gemini-pro-vision")

image_message = {
    "type": "image_url",
    "image_url": {"url": "image_example.jpg"},
}
text_message = {
    "type": "text",
    "text": "这张图片中显示了什么？",
}
message = HumanMessage(content=[text_message, image_message])

output = llm.invoke([message])
print(output.content)
```

```output
 这是一只约克夏梗。
```

让我们再次确认一下，这是一只猫吗？:)

```python
from vertexai.preview.generative_models import Image

i = Image.load_from_file("image_example.jpg")
i
```

![](/img/google_vertex_ai_palm_1.png)

您还可以将图像作为字节传递：

```python
import base64

with open("image_example.jpg", "rb") as image_file:
    image_bytes = image_file.read()

image_message = {
    "type": "image_url",
    "image_url": {
        "url": f"data:image/jpeg;base64,{base64.b64encode(image_bytes).decode('utf-8')}"
    },
}
text_message = {
    "type": "text",
    "text": "这张图片中显示了什么？",
}
message = HumanMessage(content=[text_message, image_message])

output = llm.invoke([message])
print(output.content)
```

```output
 这是一只约克夏梗。
```

请注意，您还可以使用存储在 GCS 中的图像（只需将 `url` 指向完整的 GCS 路径，以 `gs://` 开头，而不是本地路径）。

您还可以将之前聊天的历史记录传递给 LLM：

```python
message2 = HumanMessage(content="这张图片是在哪里拍摄的？")
output2 = llm.invoke([message, output, message2])
print(output2.content)
```

您还可以使用公共图像 URL：

```python
image_message = {
    "type": "image_url",
    "image_url": {
        "url": "https://python.langchain.com/assets/images/cell-18-output-1-0c7fb8b94ff032d51bfe1880d8370104.png",
    },
}
text_message = {
    "type": "text",
    "text": "这张图片中显示了什么？",
}
message = HumanMessage(content=[text_message, image_message])

output = llm.invoke([message])
print(output.content)
```

## Vertex Model Garden

Vertex Model Garden [提供](https://cloud.google.com/vertex-ai/docs/start/explore-models) 了可以在 Vertex AI 上部署和提供服务的开源模型。如果您已经成功部署了来自 Vertex Model Garden 的模型，您可以在控制台或通过 API 找到相应的 Vertex AI [端点](https://cloud.google.com/vertex-ai/docs/general/deployment#what_happens_when_you_deploy_a_model)。

```python
from langchain_google_vertexai import VertexAIModelGarden
```

```python
llm = VertexAIModelGarden(project="YOUR PROJECT", endpoint_id="YOUR ENDPOINT_ID")
```

```python
llm.invoke("生命的意义是什么？")
```

与所有 LLM 一样，我们可以将其与其他组件组合在一起：

```python
prompt = PromptTemplate.from_template("生命的意义是{thing}？")
```

```python
chain = prompt | llm
print(chain.invoke({"thing": "生活"}))
```

## 在 Vertex AI 上使用 Anthropic

> [Anthropic Claude 3](https://cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-claude) 在 Vertex AI 上提供了完全托管和无服务器的模型 API。要在 Vertex AI 上使用 Claude 模型，请直接向 Vertex AI API 端点发送请求。由于 Anthropic Claude 3 模型使用托管的 API，无需预配或管理基础架构。

注意：Anthropic Models 在 Vertex 上作为 Chat Model 实现，使用 `ChatAnthropicVertex` 类。

```python
!pip install -U langchain-google-vertexai anthropic[vertex]
```

```python
from langchain_core.messages import (
    AIMessage,
    AIMessageChunk,
    HumanMessage,
    SystemMessage,
)
from langchain_core.outputs import LLMResult
from langchain_google_vertexai.model_garden import ChatAnthropicVertex
```

注意：请指定正确的 [Claude 3 模型版本](https://cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-claude#claude-opus)
- 对于 Claude 3 Opus（预览版），请使用 `claude-3-opus@20240229`。
- 对于 Claude 3 Sonnet，请使用 `claude-3-sonnet@20240229`。
- 对于 Claude 3 Haiku，请使用 `claude-3-haiku@20240307`。

我们不建议使用不包含以 @ 符号开头的后缀的 Anthropic Claude 3 模型版本（claude-3-opus、claude-3-sonnet 或 claude-3-haiku）。

```python
# TODO：将下面的内容替换为您的项目 ID 和区域
project = "<project_id>"
location = "<region>"

# 初始化模型
model = ChatAnthropicVertex(
    model_name="claude-3-haiku@20240307",
    project=project,
    location=location,
)
```

```python
# 为模型准备输入数据
raw_context = (
    "我的名字是彼得。你是我的私人助理。我最喜欢的电影是《指环王》和《霍比特人》。"
)
question = (
    "你好，你能推荐一部今晚给我看的好电影吗？"
)
context = SystemMessage(content=raw_context)
message = HumanMessage(content=question)
```


```python
# 调用模型
response = model.invoke([context, message])
print(response.content)
```
```output
由于你最喜欢的电影是《指环王》和《霍比特人》系列，我建议你看一些其他具有类似感觉的史诗奇幻电影：

1. 《纳尼亚传奇》系列 - 这些电影改编自C.S.刘易斯的备受喜爱的奇幻小说，融合了冒险、魔法和令人难忘的角色。

2. 《星尘》 - 这部2007年的奇幻电影改编自尼尔·盖曼的小说，拥有出色的演员阵容和迷人的童话氛围。

3. 《黄金罗盘》 - 菲利普·普尔曼《黑暗元素三部曲》的首部电影改编作品，具有令人惊叹的视觉效果和引人入胜的故事。

4. 《潘的迷宫》 - 吉尔莫·德尔·托罗以西班牙内战为背景，创作了这部黑暗、受童话启发的杰作。

5. 《公主新娘》 - 一部经典的奇幻冒险电影，融合了幽默、浪漫和难忘的角色。

如果你对其中任何一部感兴趣，或者希望我推荐其他电影，请告诉我！我很乐意提供更个性化的建议。
```

```python
# 你也可以选择在调用方法中初始化/覆盖模型名称
response = model.invoke([context, message], model_name="claude-3-sonnet@20240229")
print(response.content)
```
```output
当然，我很乐意为你推荐一部电影！由于你提到《指环王》和《霍比特人》是你最喜欢的电影之一，我建议你看一些其他史诗奇幻/冒险电影，你可能会喜欢：

1. 《公主新娘》（1987年）- 一部经典的童话故事，充满冒险、浪漫和许多机智幽默。它拥有一流的演员阵容和非常经典的台词。

2. 《威洛》（1988年）- 一部由乔治·卢卡斯制作的有趣的奇幻电影，讲述了仙女、矮人和小矮人展开史诗般的冒险。它的氛围与《指环王》电影类似。

3. 《星尘》（2007年）- 这部被低估的奇幻冒险电影改编自尼尔·盖曼的小说，讲述了一个年轻人进入魔法王国寻找一颗坠落的星星的故事。拥有出色的演员阵容和视觉效果。

4. 《纳尼亚传奇》系列 - 《狮子、女巫和魔衣橱》是最著名的，但其他纳尼亚电影也是非常成功的奇幻史诗。

5. 《黄金罗盘》（2007年）-《黑暗元素三部曲》的第一部作品，设定在一个拥有装甲北极熊和寻求真相设备的平行宇宙。

如果你想要其他建议，或者有特定风格的电影想要看，请告诉我！我致力于为你推荐类似《指环王》的有趣奇幻/冒险电影。
```

```python
# 使用流式响应
sync_response = model.stream([context, message], model_name="claude-3-haiku@20240307")
for chunk in sync_response:
    print(chunk.content)
```