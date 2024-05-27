# 日志、追踪和监控

在使用 Langchain 构建应用程序或代理时，您需要进行多次 API 调用来完成单个用户请求。然而，当您想要分析这些请求时，它们并没有链接在一起。通过 [**Portkey**](/docs/integrations/providers/portkey/)，所有来自单个用户请求的嵌入、完成和其他请求都将被记录和追踪到一个公共 ID，使您能够全面了解用户的交互。

本笔记本将作为一个逐步指南，介绍如何在您的 Langchain 应用程序中使用 `Portkey` 记录、追踪和监控 Langchain LLM 调用。

首先，让我们导入 Portkey、OpenAI 和 Agent 工具

```python
import os
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_openai import ChatOpenAI
from portkey_ai import PORTKEY_GATEWAY_URL, createHeaders
```

在下面粘贴您的 OpenAI API 密钥。[(您可以在此处找到它)](https://platform.openai.com/account/api-keys)

```python
os.environ["OPENAI_API_KEY"] = "..."
```

## 获取 Portkey API 密钥

1. 在[此处](https://app.portkey.ai/signup)注册 [Portkey](https://app.portkey.ai/)

2. 在[仪表板](https://app.portkey.ai/)上，点击左下角的个人资料图标，然后点击“复制 API 密钥”

3. 在下面粘贴它

```python
PORTKEY_API_KEY = "..."  # 在此处粘贴您的 Portkey API 密钥
```

## 设置追踪 ID

1. 在下面设置您的请求的追踪 ID

2. 追踪 ID 可以对所有源自单个请求的 API 调用都是相同的

```python
TRACE_ID = "uuid-trace-id"  # 在此处设置追踪 ID
```

## 生成 Portkey Headers

```python
portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY, provider="openai", trace_id=TRACE_ID
)
```

定义提示和要使用的工具

```python
from langchain import hub
from langchain_core.tools import tool
prompt = hub.pull("hwchase17/openai-tools-agent")
@tool
def multiply(first_int: int, second_int: int) -> int:
    """Multiply two integers together."""
    return first_int * second_int
@tool
def exponentiate(base: int, exponent: int) -> int:
    "Exponentiate the base to the exponent power."
    return base**exponent
tools = [multiply, exponentiate]
```

像往常一样运行您的代理。**唯一的**变化是我们现在将**上述头部信息**包含在请求中。

```python
model = ChatOpenAI(
    base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, temperature=0
)
# 构建 OpenAI Tools 代理
agent = create_openai_tools_agent(model, tools, prompt)
# 通过传入代理和工具创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
agent_executor.invoke(
    {
        "input": "Take 3 to the fifth power and multiply that by thirty six, then square the result"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
调用: `exponentiate`，参数为 `{'base': 3, 'exponent': 5}`
243
调用: `multiply`，参数为 `{'first_int': 243, 'second_int': 36}`
8748
调用: `exponentiate`，参数为 `{'base': 8748, 'exponent': 2}`
76527504将 3 的五次方乘以 36，然后对结果进行平方的结果是 76,527,504。
> 链结束。
```

```output
{'input': 'Take 3 to the fifth power and multiply that by thirty six, then square the result',
 'output': '将 3 的五次方乘以 36，然后对结果进行平方的结果是 76,527,504。'}
```

## Portkey 上的日志和追踪工作原理

**日志**

- 通过 Portkey 发送请求可以确保默认记录所有请求

- 每个请求日志包含 `时间戳`、`模型名称`、`总成本`、`请求时间`、`请求 JSON`、`响应 JSON` 和其他 Portkey 功能

**[追踪](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/traces)**

- 追踪 ID 会随每个请求一起传递，并在 Portkey 仪表板上可见

- 如果需要，您还可以为每个请求设置一个**不同的追踪 ID**

- 您还可以将用户反馈附加到追踪 ID 上。[更多信息请参见此处](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/feedback)

对于上述请求，您将能够像这样查看整个日志追踪

![在 Portkey 上查看 Langchain 追踪](https://assets.portkey.ai/docs/agent_tracing.gif)

## 高级 LLMOps 功能 - 缓存、标记、重试

除了日志和追踪之外，Portkey 还提供了更多功能，为您现有的工作流程增加了生产能力：

**缓存**

从缓存中响应先前服务过的客户查询，而不是再次将它们发送到 OpenAI。匹配完全相同的字符串或语义相似的字符串。缓存可以节省成本，并将延迟降低 20 倍。[文档](https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations/cache-simple-and-semantic)

**重试**

自动重新处理任何不成功的 API 请求**`最多 5 次`**。使用**`指数退避`**策略，将重试尝试间隔开，以防止网络过载。[文档](https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations)

**标记**

使用预定义的标签详细跟踪和审计每个用户交互。[文档](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/metadata)