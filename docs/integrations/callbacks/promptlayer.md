# PromptLayer

>[PromptLayer](https://docs.promptlayer.com/introduction) 是一个用于提示工程的平台。它还可以帮助实现LLM的可观察性，可视化请求、版本提示，并跟踪使用情况。

>

>虽然 `PromptLayer` 确实有与LangChain直接集成的LLM（例如 [`PromptLayerOpenAI`](/docs/integrations/llms/promptlayer_openai)），但使用回调是将 `PromptLayer` 与LangChain集成的推荐方法。

在本指南中，我们将介绍如何设置 `PromptLayerCallbackHandler`。

有关更多信息，请参阅 [PromptLayer文档](https://docs.promptlayer.com/languages/langchain)。

## 安装和设置

```python
%pip install --upgrade --quiet  promptlayer --upgrade
```

### 获取API凭据

如果您没有PromptLayer帐户，请在[promptlayer.com](https://www.promptlayer.com)上创建一个。然后，通过单击导航栏中的设置齿轮来获取API密钥，并将其设置为名为 `PROMPTLAYER_API_KEY` 的环境变量。

## 用法

使用 `PromptLayerCallbackHandler` 很简单，它有两个可选参数：

1. `pl_tags` - 一个可选的字符串列表，将作为标签在PromptLayer上进行跟踪。

2. `pl_id_callback` - 一个可选的函数，以 `promptlayer_request_id` 作为参数。此ID可与PromptLayer的所有跟踪功能一起使用，以跟踪元数据、分数和提示的使用情况。

## 简单的OpenAI示例

在这个简单的示例中，我们使用 `PromptLayerCallbackHandler` 和 `ChatOpenAI`。我们添加了一个名为 `chatopenai` 的PromptLayer标签。

```python
import promptlayer  # 不要忘记这个 🍰
from langchain_community.callbacks.promptlayer_callback import (
    PromptLayerCallbackHandler,
)
```

```python
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI
chat_llm = ChatOpenAI(
    temperature=0,
    callbacks=[PromptLayerCallbackHandler(pl_tags=["chatopenai"])],
)
llm_results = chat_llm.invoke(
    [
        HumanMessage(content="1,2,3之后是什么？"),
        HumanMessage(content="再给我讲一个笑话？"),
    ]
)
print(llm_results)
```

## GPT4All示例

```python
from langchain_community.llms import GPT4All
model = GPT4All(model="./models/gpt4all-model.bin", n_ctx=512, n_threads=8)
callbacks = [PromptLayerCallbackHandler(pl_tags=["langchain", "gpt4all"])]
response = model.invoke(
    "从前有一天，",
    config={"callbacks": callbacks},
)
```

## 完整示例

在这个示例中，我们发挥了 `PromptLayer` 的更多功能。

PromptLayer允许您可视化地创建、版本化和跟踪提示模板。使用 [Prompt Registry](https://docs.promptlayer.com/features/prompt-registry)，我们可以以编程方式获取名为 `example` 的提示模板。

我们还定义了一个 `pl_id_callback` 函数，它接受 `promptlayer_request_id` 并记录分数、元数据，并链接使用的提示模板。在 [我们的文档](https://docs.promptlayer.com/features/prompt-history/request-id) 中阅读更多关于跟踪的内容。

```python
from langchain_openai import OpenAI
def pl_id_callback(promptlayer_request_id):
    print("prompt layer id ", promptlayer_request_id)
    promptlayer.track.score(
        request_id=promptlayer_request_id, score=100
    )  # 分数是一个0-100的整数
    promptlayer.track.metadata(
        request_id=promptlayer_request_id, metadata={"foo": "bar"}
    )  # 元数据是一个键值对的字典，用于在PromptLayer上进行跟踪
    promptlayer.track.prompt(
        request_id=promptlayer_request_id,
        prompt_name="example",
        prompt_input_variables={"product": "toasters"},
        version=1,
    )  # 将请求链接到提示模板
openai_llm = OpenAI(
    model_name="gpt-3.5-turbo-instruct",
    callbacks=[PromptLayerCallbackHandler(pl_id_callback=pl_id_callback)],
)
example_prompt = promptlayer.prompts.get("example", version=1, langchain=True)
openai_llm.invoke(example_prompt.format(product="toasters"))
```

就是这样！设置完成后，您的所有请求都将显示在PromptLayer仪表板上。

此回调函数还适用于在LangChain上实现的任何LLM。