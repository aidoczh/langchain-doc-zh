# Baseten

[Baseten](https://baseten.co) 是一个提供完整基础设施的服务商，可用于高效、可扩展和成本效益的部署和提供机器学习模型。

作为一个模型推理平台，`Baseten` 是 LangChain 生态系统中的一个`提供商`。`Baseten` 集成目前实现了一个单一的`组件`，即 LLMs，但计划中还会有更多！

`Baseten` 让你可以运行开源模型，比如 Llama 2 或 Mistral，并在专用 GPU 上运行专有或经过精细调整的模型。如果你习惯于像 OpenAI 这样的服务商，使用 Baseten 会有一些不同之处：

* 你不是按 token 支付，而是按 GPU 使用的分钟数支付。

* Baseten 上的每个模型都使用我们的开源模型打包框架 [Truss](https://truss.baseten.co/welcome)，以实现最大的可定制性。

* 虽然我们有一些与 [OpenAI ChatCompletions 兼容的模型](https://docs.baseten.co/api-reference/openai)，你也可以使用 `Truss` 定义自己的 I/O 规范。

[了解更多](https://docs.baseten.co/deploy/lifecycle) 关于模型 ID 和部署。

在 [Baseten 文档](https://docs.baseten.co/) 中了解更多关于 Baseten 的信息。

## 安装和设置

要在 LangChain 中使用 Baseten 模型，你需要两样东西：

- 一个 [Baseten 账户](https://baseten.co)

- 一个 [API 密钥](https://docs.baseten.co/observability/api-keys)

将你的 API 密钥导出为一个名为 `BASETEN_API_KEY` 的环境变量。

```sh
export BASETEN_API_KEY="在这里粘贴你的 API 密钥"
```

## LLMs

查看 [使用示例](/docs/integrations/llms/baseten)。

```python
from langchain_community.llms import Baseten
```