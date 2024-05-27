

# NVIDIA AI Foundation Endpoints

`ChatNVIDIA` 类是一个 LangChain 聊天模型，连接到 [NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/)。

> [NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/) 为用户提供了访问 NVIDIA 托管的 API 端点的便利，其中包括 Mixtral 8x7B、Llama 2、Stable Diffusion 等 NVIDIA AI Foundation 模型。这些模型托管在 [NVIDIA API 目录](https://build.nvidia.com/) 上，经过优化、测试并托管在 NVIDIA AI 平台上，使其快速且易于评估，进一步定制，并在任何加速堆栈上以最佳性能无缝运行。

> 

> 使用 [NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/)，您可以从在 [NVIDIA DGX Cloud](https://www.nvidia.com/en-us/data-center/dgx-cloud/) 上运行的完全加速堆栈中快速获得结果。一旦定制完成，这些模型可以在具有企业级安全性、稳定性和支持的情况下部署到任何地方，使用 [NVIDIA AI Enterprise](https://www.nvidia.com/en-us/data-center/products/ai-enterprise/)。

> 

> 这些模型可以通过 [`langchain-nvidia-ai-endpoints`](https://pypi.org/project/langchain-nvidia-ai-endpoints/) 软件包轻松访问，如下所示。

这个示例演示了如何使用 LangChain 与公开可访问的 AI Foundation 端点进行交互和开发基于 LLM 的系统。

## 安装

```python
%pip install --upgrade --quiet langchain-nvidia-ai-endpoints
```

## 设置

**开始使用：**

1. 在 [NVIDIA](https://build.nvidia.com/) 上创建一个免费帐户，该网站托管 NVIDIA AI Foundation 模型。

2. 点击您选择的模型。

3. 在 `Input` 下选择 `Python` 选项卡，然后点击 `Get API Key`。接着点击 `Generate Key`。

4. 复制并保存生成的密钥为 `NVIDIA_API_KEY`。从那里，您将可以访问这些端点。

```python
import getpass
import os
if not os.environ.get("NVIDIA_API_KEY", "").startswith("nvapi-"):
    nvapi_key = getpass.getpass("Enter your NVIDIA API key: ")
    assert nvapi_key.startswith("nvapi-"), f"{nvapi_key[:5]}... is not a valid key"
    os.environ["NVIDIA_API_KEY"] = nvapi_key
```
```python
## 核心 LC 聊天界面
from langchain_nvidia_ai_endpoints import ChatNVIDIA
llm = ChatNVIDIA(model="mistralai/mixtral-8x7b-instruct-v0.1")
result = llm.invoke("Write a ballad about LangChain.")
print(result.content)
```

## 流式处理、批处理和异步处理

这些模型原生支持流式处理，与所有 LangChain LLM 一样，它们提供了一个批处理方法来处理并发请求，以及用于调用、流式处理和批处理的异步方法。以下是一些示例。

```python
print(llm.batch(["What's 2*3?", "What's 2*6?"]))
# 或者通过异步 API
# await llm.abatch(["What's 2*3?", "What's 2*6?"])
```
```python
for chunk in llm.stream("How far can a seagull fly in one day?"):
    # 显示标记分隔
    print(chunk.content, end="|")
```
```python
async for chunk in llm.astream("How long does it take for monarch butterflies to migrate?"):
    print(chunk.content, end="|")
```

## 支持的模型

查询 `available_models` 仍将为您提供 API 凭据提供的所有其他模型。

`playground_` 前缀是可选的。

```python
ChatNVIDIA.get_available_models()
# llm.get_available_models()
```

## 模型类型

所有上述模型都受支持，并可以通过 `ChatNVIDIA` 访问。

某些模型类型支持独特的提示技术和聊天消息。我们将在下面回顾一些重要的模型。

**要了解有关特定模型的更多信息，请导航至 AI Foundation 模型的 API 部分 [链接在此处](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/codellama-13b/api)。**

### 通用聊天

诸如 `meta/llama3-8b-instruct` 和 `mistralai/mixtral-8x22b-instruct-v0.1` 等模型是良好的全能模型，您可以将其用于任何 LangChain 聊天消息。以下是一个示例。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_nvidia_ai_endpoints import ChatNVIDIA
prompt = ChatPromptTemplate.from_messages(
    [("system", "You are a helpful AI assistant named Fred."), ("user", "{input}")]
)
chain = prompt | ChatNVIDIA(model="meta/llama3-8b-instruct") | StrOutputParser()
for txt in chain.stream({"input": "What's your name?"}):
    print(txt, end="")
```

### 代码生成

这些模型接受与常规聊天模型相同的参数和输入结构，但它们在代码生成和结构化代码任务上表现更好。其中一个示例是 `meta/codellama-70b`。

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert coding AI. Respond only in valid python; no narration whatsoever.",
        ),
        ("user", "{input}"),
    ]
)
chain = prompt | ChatNVIDIA(model="meta/codellama-70b") | StrOutputParser()
for txt in chain.stream({"input": "How do I solve this fizz buzz problem?"}):
    print(txt, end="")
## LLM模型的动态调整
[SteerLM-optimized models](https://developer.nvidia.com/blog/announcing-steerlm-a-simple-and-practical-technique-to-customize-llms-during-inference/) 支持在推理时对模型输出进行“动态调整”。
这使您可以通过在0到9的整数标签上控制模型的复杂性、冗长性和创造力。在底层，这些标签被传递为一种特殊类型的助手消息给模型。
“steer”模型支持这种类型的输入，比如 `nemotron_steerlm_8b`。
```python

from langchain_nvidia_ai_endpoints import ChatNVIDIA

llm = ChatNVIDIA(model="nemotron_steerlm_8b")

# 尝试让它不创造性且不啰嗦

complex_result = llm.invoke(

    "What's a PB&J?", labels={"creativity": 0, "complexity": 3, "verbosity": 0}

)

print("不创造性\n")

print(complex_result.content)

# 尝试让它非常有创造性和啰嗦

print("\n\n有创造性\n")

creative_result = llm.invoke(

    "What's a PB&J?", labels={"creativity": 9, "complexity": 3, "verbosity": 9}

)

print(creative_result.content)

```
#### 在LCEL中的使用
标签作为调用参数传递。您可以使用LLM上的`bind`方法将其绑定到LCEL中，以在声明性、功能链中包含它。以下是一个示例。
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_nvidia_ai_endpoints import ChatNVIDIA
prompt = ChatPromptTemplate.from_messages(
    [("system", "You are a helpful AI assistant named Fred."), ("user", "{input}")]
)
chain = (
    prompt
    | ChatNVIDIA(model="nemotron_steerlm_8b").bind(
        labels={"creativity": 9, "complexity": 0, "verbosity": 9}
    )
    | StrOutputParser()
)
for txt in chain.stream({"input": "Why is a PB&J?"}):
    print(txt, end="")
```
## 多模态
NVIDIA还支持多模态输入，这意味着您可以为模型提供图像和文本进行推理。支持多模态输入的示例模型是 `playground_neva_22b`。
这些模型接受LangChain的标准图像格式，并接受类似于上述Steering LLM的`labels`。除了`creativity`、`complexity`和`verbosity`之外，这些模型还支持`quality`切换。
以下是一个使用示例。
```python
import IPython
import requests
image_url = "https://www.nvidia.com/content/dam/en-zz/Solutions/research/ai-playground/nvidia-picasso-3c33-p@2x.jpg"  ## 大图像
image_content = requests.get(image_url).content
IPython.display.Image(image_content)
```
```python
from langchain_nvidia_ai_endpoints import ChatNVIDIA
llm = ChatNVIDIA(model="playground_neva_22b")
```
#### 作为URL传递图像
```python
from langchain_core.messages import HumanMessage
llm.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "Describe this image:"},
                {"type": "image_url", "image_url": {"url": image_url}},
            ]
        )
    ]
)
```
#### 作为base64编码字符串传递图像
目前，客户端会进行一些额外处理以支持像上面那样的大图像。但对于较小的图像（为了更好地说明底层的处理过程），我们可以直接传入图像，如下所示：
```python
import base64
from langchain_core.messages import HumanMessage
## 对于更简单的图像有效。对于较大的图像，请参见实际实现
b64_string = base64.b64encode(image_content).decode("utf-8")
llm.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "Describe this image:"},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/png;base64,{b64_string}"},
                },
            ]
        )
    ]
)
```
#### 直接在字符串中
NVIDIA API独特地接受图像作为内联在`<img/>` HTML标签中的base64图像。虽然这不太与其他LLM兼容，但您可以直接相应地提示模型。
```python
base64_with_mime_type = f"data:image/png;base64,{b64_string}"
llm.invoke(f'What\'s in this image?\n<img src="{base64_with_mime_type}" />')
```
#### **高级用例：** 强制载荷
您可能注意到一些新模型可能具有强烈的参数期望，而LangChain连接器可能默认不支持。例如，由于服务器端缺少流参数，我们在此笔记本的最新版本发布时无法调用[Kosmos](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/kosmos-2)模型：```
```markdown
从 langchain_nvidia_ai_endpoints 导入 ChatNVIDIA
kosmos = ChatNVIDIA(model="kosmos_2")
从 langchain_core.messages 导入 HumanMessage
# kosmos.invoke(
#     [
#         HumanMessage(
#             content=[
#                 {"type": "text", "text": "Describe this image:"},
#                 {"type": "image_url", "image_url": {"url": image_url}},
#             ]
#         )
#     ]
# )
# 异常: [422] 无法处理的实体
# body -> stream
#   不允许额外输入 (type=extra_forbidden)
# 请求ID: 35538c9a-4b45-4616-8b75-7ef816fccf38
```

对于这样一个简单的用例，我们实际上可以通过指定 `payload_fn` 函数来强制底层客户端的 payload 参数，如下所示：

```python
def drop_streaming_key(d):
    """接受 payload 字典，输出新的 payload 字典"""
    if "stream" in d:
        d.pop("stream")
    return d
## 覆盖 payload 传递。默认情况下是按原样传递 payload。
kosmos = ChatNVIDIA(model="kosmos_2")
kosmos.client.payload_fn = drop_streaming_key
kosmos.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "Describe this image:"},
                {"type": "image_url", "image_url": {"url": image_url}},
            ]
        )
    ]
)
```

对于更高级或自定义的用例（即支持扩散模型），您可能有兴趣利用 `NVEModel` 客户端作为请求的支撑。`NVIDIAEmbeddings` 类是此操作的良好灵感来源。

## RAG：上下文模型

NVIDIA 还拥有支持包含检索到的上下文的特殊 "context" 聊天消息的问答模型（例如 RAG 链中的文档）。这对于避免在模型中注入提示非常有用。像 `nemotron_qa_8b` 这样的 `_qa_` 模型支持这一点。

**注意：** 仅支持这些模型的 "user"（人类）和 "context" 聊天消息；不支持在对话流程中有用的系统或 AI 消息。

```python
从 langchain_core.messages 导入 ChatMessage
从 langchain_core.output_parsers 导入 StrOutputParser
从 langchain_core.prompts 导入 ChatPromptTemplate
从 langchain_nvidia_ai_endpoints 导入 ChatNVIDIA
prompt = ChatPromptTemplate.from_messages(
    [
        ChatMessage(
            role="context", content="Parrots and Cats have signed the peace accord."
        ),
        ("user", "{input}"),
    ]
)
llm = ChatNVIDIA(model="nemotron_qa_8b")
chain = prompt | llm | StrOutputParser()
chain.invoke({"input": "What was signed?"})
```

## 在对话链中的示例用法

与任何其他集成一样，ChatNVIDIA 默认支持对话缓冲区等聊天实用程序。下面，我们展示了应用于 `mistralai/mixtral-8x22b-instruct-v0.1` 模型的 [LangChain ConversationBufferMemory](https://python.langchain.com/docs/modules/memory/types/buffer) 示例。

```python
%pip install --upgrade --quiet langchain
```
```python
从 langchain.chains 导入 ConversationChain
从 langchain.memory 导入 ConversationBufferMemory
chat = ChatNVIDIA(
    model="mistralai/mixtral-8x22b-instruct-v0.1",
    temperature=0.1,
    max_tokens=100,
    top_p=1.0,
)
conversation = ConversationChain(llm=chat, memory=ConversationBufferMemory())
```
```python
conversation.invoke("Hi there!")["response"]
```
```python
conversation.invoke("I'm doing well! Just having a conversation with an AI.")[
    "response"
]
```
```python
conversation.invoke("Tell me about yourself.")["response"]
```
