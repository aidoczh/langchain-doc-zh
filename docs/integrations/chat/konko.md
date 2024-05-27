# ChatKonko

# Konko

>[Konko](https://www.konko.ai/) API 是一个完全托管的 Web API，旨在帮助应用程序开发人员：

1. **选择**合适的开源或专有的 LLM 模型用于他们的应用程序

2. 通过与主要应用程序框架和完全托管的 API 集成，**更快地构建**应用程序

3. **微调**较小的开源 LLM 模型，以在成本的一小部分下实现行业领先的性能

4. 使用 Konko AI 的符合 SOC 2 标准的多云基础设施，**部署符合安全性、隐私性、吞吐量和延迟 SLA** 的生产规模 API，无需设置或管理基础设施

这个示例介绍了如何使用 LangChain 与 `Konko` 的 ChatCompletion [模型](https://docs.konko.ai/docs/list-of-models#konko-hosted-models-for-chatcompletion) 进行交互。

要运行这个笔记本，您需要 Konko API 密钥。登录到我们的 Web 应用程序，[创建一个 API 密钥](https://platform.konko.ai/settings/api-keys) 来访问模型。

```python
from langchain_community.chat_models import ChatKonko
from langchain_core.messages import HumanMessage, SystemMessage
```

#### 设置环境变量

1. 您可以为以下环境变量设置值：

   1. KONKO_API_KEY（必需）

   2. OPENAI_API_KEY（可选）

2. 在当前的 shell 会话中，使用 export 命令：

```shell
export KONKO_API_KEY={your_KONKO_API_KEY_here}
export OPENAI_API_KEY={your_OPENAI_API_KEY_here} #可选
```

## 调用模型

在 [Konko 概览页面](https://docs.konko.ai/docs/list-of-models) 找到一个模型。

另一种查找在 Konko 实例上运行的模型列表的方法是通过这个 [端点](https://docs.konko.ai/reference/get-models)。

从这里，我们可以初始化我们的模型：

```python
chat = ChatKonko(max_tokens=400, model="meta-llama/llama-2-13b-chat")
```

```python
messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Explain Big Bang Theory briefly"),
]
chat(messages)
```

```output
AIMessage(content="  当然！宇宙大爆炸理论是一个解释宇宙起源的科学理论。简而言之，它表明宇宙在约 138 亿年前开始作为一个无限炽热和密集的点，并迅速膨胀。这种膨胀持续至今，这也是使宇宙看起来如此的原因。\n\n以下是关键点的简要概述：\n\n1. 宇宙始于奇点，一个无限密度和温度的点。\n2. 奇点迅速膨胀，导致宇宙冷却和膨胀。\n3. 随着宇宙的膨胀，开始形成粒子，包括质子、中子和电子。\n4. 这些粒子最终聚集在一起形成了原子，后来形成了恒星和星系。\n5. 宇宙至今仍在膨胀，而这种膨胀的速度正在加快。\n\n这就是宇宙大爆炸理论的要点！当您考虑到这一点时，这是一个非常令人惊叹的想法，并且得到了大量的科学证据的支持。您对此还有其他问题吗？")
```