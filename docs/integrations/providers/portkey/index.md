# 门户

[门户](https://portkey.ai) 是 AI 应用的控制面板。凭借其流行的 AI 网关和可观察性套件，数百个团队发布**可靠**、**高效**和**快速**的应用程序。

## Langchain 的 LLMOps

门户为 Langchain 带来了生产就绪性。使用门户，您可以：

- [x] 通过统一 API 连接到 150 多个模型，

- [x] 查看所有请求的 42 个**指标和日志**，

- [x] 启用**语义缓存**以减少延迟和成本，

- [x] 为失败的请求实现自动**重试和回退**，

- [x] 为请求添加**自定义标签**以进行更好的跟踪和分析以及[更多](https://portkey.ai/docs)。

## 快速入门 - 门户和 Langchain

由于门户与 OpenAI 签名完全兼容，您可以通过 `ChatOpenAI` 接口连接到门户 AI 网关。

- 将 `base_url` 设置为 `PORTKEY_GATEWAY_URL`

- 使用 `createHeaders` 辅助方法添加 `default_headers` 以使用门户所需的标头。

首先，通过[此处注册](https://app.portkey.ai/signup)获取您的门户 API 密钥（单击左下角的个人资料图标，然后单击“复制 API 密钥”），或在[您自己的环境](https://github.com/Portkey-AI/gateway/blob/main/docs/installation-deployments.md)中部署开源 AI 网关。

接下来，安装门户 SDK

```python
pip install -U portkey_ai
```

现在，我们可以通过更新 Langchain 中的 `ChatOpenAI` 模型连接到门户 AI 网关

```python
from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders, PORTKEY_GATEWAY_URL
PORTKEY_API_KEY = "..." # 在托管自己的网关时不需要
PROVIDER_API_KEY = "..." # 添加所使用的 AI 提供商的 API 密钥
portkey_headers = createHeaders(api_key=PORTKEY_API_KEY,provider="openai")
llm = ChatOpenAI(api_key=PROVIDER_API_KEY, base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers)
llm.invoke("生命、宇宙和一切的意义是什么？")
```

该请求通过您的门户 AI 网关路由到指定的`提供商`。门户还将开始记录所有请求，使调试变得非常简单。

![在门户中查看来自 Langchain 的日志](https://assets.portkey.ai/docs/langchain-logs.gif)

## 通过 AI 网关使用 150 多个模型

当您能够使用上述代码片段连接到 20 多个提供商支持的 150 多个模型时，AI 网关的强大功能就显现出来了。

让我们修改上面的代码，以调用 Anthropic 的 `claude-3-opus-20240229` 模型。

门户支持**[虚拟密钥](https://docs.portkey.ai/docs/product/ai-gateway-streamline-llm-integrations/virtual-keys)**，这是一种在安全保险库中存储和管理 API 密钥的简便方法。让我们尝试使用虚拟密钥进行 LLM 调用。您可以转到门户中的虚拟密钥选项卡，并为 Anthropic 创建一个新密钥。

`virtual_key` 参数设置了所使用的身份验证和提供商。在我们的情况下，我们使用了 Anthropic 的虚拟密钥。

> 请注意，`api_key` 可以留空，因为不会使用该身份验证。

```python
from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders, PORTKEY_GATEWAY_URL
PORTKEY_API_KEY = "..."
VIRTUAL_KEY = "..." # 我们上面复制的 Anthropic 虚拟密钥
portkey_headers = createHeaders(api_key=PORTKEY_API_KEY,virtual_key=VIRTUAL_KEY)
llm = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, model="claude-3-opus-20240229")
llm.invoke("生命、宇宙和一切的意义是什么？")
```

门户 AI 网关将对 Anthropic 的 API 请求进行身份验证，并以 OpenAI 格式返回响应供您使用。

AI 网关扩展了 Langchain 的 `ChatOpenAI` 类，使其成为调用任何提供商和任何模型的单一接口。

## 高级路由 - 负载均衡、回退、重试

门户 AI 网关通过基于配置的方法为 Langchain 带来了负载均衡、回退、实验和金丝雀测试等功能。

让我们以一个**示例**为例，假设我们希望在 `gpt-4` 和 `claude-opus` 之间以 50:50 的比例分配流量，以测试这两个大型模型。此配置的网关配置如下所示：

```python
config = {
    "strategy": {
         "mode": "loadbalance"
    },
    "targets": [{
        "virtual_key": "openai-25654", # OpenAI 的虚拟密钥
        "override_params": {"model": "gpt4"},
        "weight": 0.5
    }, {
        "virtual_key": "anthropic-25654", # Anthropic 的虚拟密钥
        "override_params": {"model": "claude-3-opus-20240229"},
        "weight": 0.5
    }]
}
```

然后，我们可以在从 Langchain 发出的请求中使用此配置。

```python
portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY,
    config=config
)
llm = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers)
llm.invoke("生命、宇宙和一切的意义是什么？")
```

当调用 LLM 时，Portkey 将按照定义的权重比例将请求分发到 `gpt-4` 和 `claude-3-opus-20240229`。

您可以在[这里](https://docs.portkey.ai/docs/api-reference/config-object#examples)找到更多配置示例。

## **追踪链与代理**

Portkey 的 Langchain 集成使您完全能够查看代理的运行情况。让我们以一个[常见的代理工作流程](https://python.langchain.com/docs/use_cases/tool_use/quickstart/#agents)为例。

我们只需要修改 `ChatOpenAI` 类以使用上述的 AI 网关。

```python
from langchain import hub  
from langchain.agents import AgentExecutor, create_openai_tools_agent  
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from portkey_ai import PORTKEY_GATEWAY_URL, createHeaders
prompt = hub.pull("hwchase17/openai-tools-agent")
portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY,
    virtual_key=OPENAI_VIRTUAL_KEY,
    trace_id="uuid-uuid-uuid-uuid"
)
@tool
def multiply(first_int: int, second_int: int) -> int:
    """将两个整数相乘。"""
    return first_int * second_int
@tool  
def exponentiate(base: int, exponent: int) -> int:  
    "将底数的指数幂。"  
    return base**exponent  
tools = [multiply, exponentiate]
model = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, temperature=0)
# 构建 OpenAI 工具代理  
agent = create_openai_tools_agent(model, tools, prompt)
# 通过传入代理和工具来创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
agent_executor.invoke({
    "input": "将 3 的五次方乘以三十六，然后对结果求平方"
})
```

**您可以在 Portkey 仪表板上查看请求日志以及追踪 ID：**

![Langchain Agent Logs on Portkey](https://assets.portkey.ai/docs/agent_tracing.gif)

其他文档可在以下链接找到：

- 可观测性 - https://portkey.ai/docs/product/observability-modern-monitoring-for-llms

- AI 网关 - https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations

- 提示库 - https://portkey.ai/docs/product/prompt-library

您可以在此处查看我们流行的开源 AI 网关 - https://github.com/portkey-ai/gateway

有关每个功能的详细信息以及如何使用，请参阅[Portkey 文档](https://portkey.ai/docs)。如果您有任何问题或需要进一步协助，请在 [Twitter 上联系我们](https://twitter.com/portkeyai) 或发送邮件至 [support email](mailto:hello@portkey.ai)。