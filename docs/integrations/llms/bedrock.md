# 基岩

[亚马逊基岩](https://aws.amazon.com/bedrock/)是一项完全托管的服务，提供了来自领先人工智能公司如`AI21 Labs`、`Anthropic`、`Cohere`、`Meta`、`Stability AI`和`Amazon`的高性能基础模型（FMs），通过单个 API，以及一系列广泛的功能，帮助您构建具有安全性、隐私性和负责任人工智能的生成式人工智能应用。使用`亚马逊基岩`，您可以轻松尝试和评估适用于您的用例的顶级 FMs，使用诸如微调和`检索增强生成`（`RAG`）等技术私密定制它们，并构建能够使用企业系统和数据源执行任务的代理。由于`亚马逊基岩`是无服务器的，您无需管理任何基础设施，可以安全地集成和部署生成式人工智能功能到您已熟悉的 AWS 服务中。

```python
%pip install --upgrade --quiet  boto3
```

```python
from langchain_community.llms import Bedrock
llm = Bedrock(
    credentials_profile_name="bedrock-admin", model_id="amazon.titan-text-express-v1"
)
```

### 在对话链中使用

```python
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
conversation = ConversationChain(
    llm=llm, verbose=True, memory=ConversationBufferMemory()
)
conversation.predict(input="Hi there!")
```

### 带有流式处理的对话链

```python
from langchain_community.llms import Bedrock
from langchain_core.callbacks import StreamingStdOutCallbackHandler
llm = Bedrock(
    credentials_profile_name="bedrock-admin",
    model_id="amazon.titan-text-express-v1",
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
```

```python
conversation = ConversationChain(
    llm=llm, verbose=True, memory=ConversationBufferMemory()
)
conversation.predict(input="Hi there!")
```

### 自定义模型

```python
custom_llm = Bedrock(
    credentials_profile_name="bedrock-admin",
    provider="cohere",
    model_id="<Custom model ARN>",  # ARN like 'arn:aws:bedrock:...' obtained via provisioning the custom model
    model_kwargs={"temperature": 1},
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
conversation = ConversationChain(
    llm=custom_llm, verbose=True, memory=ConversationBufferMemory()
)
conversation.predict(input="What is the recipe of mayonnaise?")
```

### 亚马逊基岩示例的防护措施

## 亚马逊基岩的防护措施（预览版）

[亚马逊基岩的防护措施](https://aws.amazon.com/bedrock/guardrails/)根据用例特定策略评估用户输入和模型响应，并提供了一个额外的安全层，无论基础模型如何。防护措施可以应用于包括Anthropic Claude、Meta Llama 2、Cohere Command、AI21 Labs Jurassic和Amazon Titan Text在内的各种模型，以及经过微调的模型。

**注意**：亚马逊基岩的防护措施目前处于预览阶段，尚未普遍提供。如果您希望访问此功能，请通过您通常的 AWS 支持联系人联系。在本节中，我们将设置一个带有特定防护措施的基岩语言模型，其中包括追踪功能。

```python
from typing import Any
from langchain_core.callbacks import AsyncCallbackHandler
class BedrockAsyncCallbackHandler(AsyncCallbackHandler):
    # 可用于处理来自 langchain 的回调的异步回调处理程序。
    async def on_llm_error(self, error: BaseException, **kwargs: Any) -> Any:
        reason = kwargs.get("reason")
        if reason == "GUARDRAIL_INTERVENED":
            print(f"Guardrails: {kwargs}")
# 带有追踪功能的亚马逊基岩防护措施
llm = Bedrock(
    credentials_profile_name="bedrock-admin",
    model_id="<Model_ID>",
    model_kwargs={},
    guardrails={"id": "<Guardrail_ID>", "version": "<Version>", "trace": True},
    callbacks=[BedrockAsyncCallbackHandler()],
)
```