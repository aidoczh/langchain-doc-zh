---

sidebar_label: Konko

---

# Konko

>[Konko](https://www.konko.ai/) API 是一个完全托管的 Web API，旨在帮助应用程序开发人员：

1. **选择**合适的开源或专有 LLMs 用于他们的应用程序

2. 通过与主要应用程序框架和完全托管的 API 集成，**更快地构建**应用程序

3. **微调**较小的开源 LLMs，以实现行业领先的性能，成本仅为一小部分

4. 使用 Konko AI 符合 SOC 2 标准的多云基础设施，**部署符合安全性、隐私性、吞吐量和延迟 SLA** 的生产规模 API，无需设置或管理基础设施

此示例介绍了如何使用 LangChain 与 `Konko` 完整性 [模型](https://docs.konko.ai/docs/list-of-models#konko-hosted-models-for-completion) 进行交互

要运行此笔记本，您将需要 Konko API 密钥。请登录到我们的 Web 应用程序 [创建 API 密钥](https://platform.konko.ai/settings/api-keys) 以访问模型

#### 设置环境变量

1. 您可以为以下环境变量设置值

   1. KONKO_API_KEY（必需）

   2. OPENAI_API_KEY（可选）

2. 在当前的 shell 会话中，使用 export 命令：

```shell
export KONKO_API_KEY={your_KONKO_API_KEY_here}
export OPENAI_API_KEY={your_OPENAI_API_KEY_here} #可选
```

## 调用模型

在 [Konko 概览页面](https://docs.konko.ai/docs/list-of-models) 上找到一个模型

另一种查找在 Konko 实例上运行的模型列表的方法是通过这个 [端点](https://docs.konko.ai/reference/get-models)。

从这里，我们可以初始化我们的模型：

```python
from langchain.llms import Konko
llm = Konko(model="mistralai/mistral-7b-v0.1", temperature=0.1, max_tokens=128)
input_ = """You are a helpful assistant. Explain Big Bang Theory briefly."""
print(llm.invoke(input_))
```

```output
答案:
宇宙大爆炸理论是一种解释宇宙起源的理论。根据这个理论，宇宙始于一个无限密度和温度的单一点。这一点被称为奇点。奇点爆炸并迅速膨胀。宇宙的膨胀仍在继续。
宇宙大爆炸理论是一种解释宇宙起源的理论。根据这个理论，宇宙始于一个无限密度和温度的单一点。这一点被称为奇点。奇点爆炸并迅速膨胀。宇宙的膨胀仍在继续。
问题
```