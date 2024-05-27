---

sidebar_label: Bedrock

---

# ChatBedrock

>[Amazon Bedrock](https://aws.amazon.com/bedrock/) 是一项完全托管的服务，提供了来自领先人工智能公司如 `AI21 Labs`, `Anthropic`, `Cohere`, `Meta`, `Stability AI` 和 `Amazon` 的高性能基础模型（FMs）选择，通过单一 API 提供了构建生成式人工智能应用所需的广泛功能，包括安全性、隐私性和负责任的人工智能。使用 `Amazon Bedrock`，您可以轻松地尝试和评估适合您用例的顶级 FMs，使用诸如微调和 `Retrieval Augmented Generation` (`RAG`) 等技术，私密地定制这些模型，并构建能够使用企业系统和数据源执行任务的代理程序。由于 `Amazon Bedrock` 是无服务器的，您无需管理任何基础架构，并且可以使用您已经熟悉的 AWS 服务安全地集成和部署生成式人工智能功能到您的应用程序中。

```python
%pip install --upgrade --quiet  langchain-aws
```

```output
Note: you may need to restart the kernel to use updated packages.
```

```python
from langchain_aws import ChatBedrock
from langchain_core.messages import HumanMessage
```

```python
chat = ChatBedrock(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",
    model_kwargs={"temperature": 0.1},
)
```

```python
messages = [
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    )
]
chat.invoke(messages)
```

```output
AIMessage(content="Voici la traduction en français :\n\nJ'aime la programmation.", additional_kwargs={'usage': {'prompt_tokens': 20, 'completion_tokens': 21, 'total_tokens': 41}}, response_metadata={'model_id': 'anthropic.claude-3-sonnet-20240229-v1:0', 'usage': {'prompt_tokens': 20, 'completion_tokens': 21, 'total_tokens': 41}}, id='run-994f0362-0e50-4524-afad-3c4f5bb11328-0')
```

### 流式处理

要进行流式处理，您可以使用可运行的 `.stream()` 方法。

```python
for chunk in chat.stream(messages):
    print(chunk.content, end="", flush=True)
```

```output
Voici la traduction en français :
J'aime la programmation.
```