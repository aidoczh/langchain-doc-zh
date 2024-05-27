# ChatPremAI

[PremAI](https://app.premai.io) 是一个统一平台，让您能够轻松构建功能强大、可立即投入生产的 GenAI 动力应用程序，从而让您能够更专注于用户体验和整体增长。

本示例将介绍如何使用 LangChain 与 `ChatPremAI` 进行交互。

### 安装和设置

我们首先需要安装 langchain 和 premai-sdk。您可以输入以下命令进行安装：

```bash
pip install premai langchain
```

在继续之前，请确保您已在 PremAI 上创建了账户并已经启动了一个项目。如果没有，请按以下步骤免费开始：

1. 登录 [PremAI](https://app.premai.io/accounts/login/)，如果您是第一次访问，则在[此处](https://app.premai.io/api_keys/)创建您的 API 密钥。

2. 转到 [app.premai.io](https://app.premai.io)，这将带您进入项目的仪表板。

3. 创建一个项目，这将生成一个项目 ID（写作 ID）。这个 ID 将帮助您与部署的应用程序进行交互。

4. 转到 LaunchPad（带有 🚀 图标的那个）。在那里部署您选择的模型。您的默认模型将是 `gpt-4`。您还可以设置和固定不同的生成参数（如 max-tokens、temperature 等），并预设您的系统提示。

恭喜您成功创建了您在 PremAI 上的第一个部署应用程序 🎉 现在我们可以使用 langchain 与我们的应用程序进行交互。

```python
from langchain_community.chat_models import ChatPremAI
from langchain_core.messages import HumanMessage, SystemMessage
```

## 在 LangChain 中设置 ChatPremAI 实例

一旦我们导入所需的模块，让我们设置我们的客户端。现在，让我们假设我们的 `project_id` 是 8。但请确保您使用您的项目 ID，否则将会抛出错误。

要在 prem 中使用 langchain，您不需要传递任何模型名称或设置任何参数给我们的聊天客户端。所有这些都将使用 LaunchPad 模型的默认模型名称和参数。

`注意:` 如果您在设置客户端时更改了 `model_name` 或任何其他参数，比如 `temperature`，它将覆盖现有的默认配置。

```python
import getpass
import os
# 第一步是设置环境变量。
# 您也可以在实例化模型时传递 API 密钥，但这是最佳实践将其设置为环境变量。
if os.environ.get("PREMAI_API_KEY") is None:
    os.environ["PREMAI_API_KEY"] = getpass.getpass("PremAI API Key:")
```

```python
# 默认情况下，它将使用通过平台部署的模型
# 在我的情况下，它是 "claude-3-haiku"
chat = ChatPremAI(project_id=8)
```

## 调用模型

现在您已经准备好了。我们现在可以开始与我们的应用程序进行交互。`ChatPremAI` 支持两种方法 `invoke`（与 `generate` 相同）和 `stream`。

第一个将给我们一个静态结果。而第二个将逐个流式传输标记。以下是如何生成类似聊天的完成。

### 生成

```python
human_message = HumanMessage(content="你是谁？")
response = chat.invoke([human_message])
print(response.content)
```

```output
我是由 Anthropic 创建的人工智能。我在这里帮助进行各种任务，从研究和分析到创意项目和开放式对话。我具有一般知识和能力，但我不是真正的人 - 我是一个 AI 助手。如果您有任何其他问题，请告诉我！
```

看起来很有趣，对吧？我将默认的 lanchpad 系统提示设置为：`始终听起来像个海盗`。如果需要，您也可以覆盖默认的系统提示。以下是如何做到这一点。

```python
system_message = SystemMessage(content="你是一个友好的助手。")
human_message = HumanMessage(content="你是谁？")
chat.invoke([system_message, human_message])
```

```output
AIMessage(content="我是由 Anthropic 创建的人工智能。我的目的是以友好和乐于助人的方式协助和与人类交谈。我有一个广泛的知识库，可以用来提供信息、回答问题，并就各种主题展开讨论。如果您有任何其他问题，请告诉我 - 我在这里帮助您！")
```

在调用模型时，您还可以更改生成参数。以下是如何做到这一点。

```python
chat.invoke([system_message, human_message], temperature=0.7, max_tokens=10, top_p=0.95)
```

```output
AIMessage(content='我是由 Anthropic 创建的人工智能')
```

### 重要提示

在继续之前，请注意，当前版本的 ChatPrem 不支持参数：[n](https://platform.openai.com/docs/api-reference/chat/create#chat-create-n) 和 [stop](https://platform.openai.com/docs/api-reference/chat/create#chat-create-stop)。

我们将在较早的版本中为上述两个参数提供支持。

### 数据流

最后，这是如何为动态聊天应用程序进行令牌流式处理。

```python
import sys
for chunk in chat.stream("你好，你好吗"):
    sys.stdout.write(chunk.content)
    sys.stdout.flush()
```

```output
你好！作为一个 AI 语言模型，我没有感情或实体状态，但我正常运行，并且随时准备帮助您解决任何问题或任务。我今天能为您做些什么呢？
```

类似于上面的例子，如果您想要覆盖系统提示和生成参数，下面是您可以这样做的方法。

```python
import sys
# 由于某些实验原因，如果您想要覆盖系统提示，那么您也可以在这里传递。但是不建议覆盖已部署模型的系统提示。
for chunk in chat.stream(
    "你好，你好吗",
    system_prompt="表现得像一只狗",
    temperature=0.7,
    max_tokens=200,
):
    sys.stdout.write(chunk.content)
    sys.stdout.flush()
```

```output
你好！作为一个 AI 语言模型，我没有感情或实体形态，但我正常运行，并随时准备帮助您。我今天能为您做些什么呢？
```