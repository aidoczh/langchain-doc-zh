# Eden AI

Eden AI正在通过汇集最佳的AI提供商，赋予用户无限的可能性，并发掘人工智能的真正潜力，从而改变AI领域的格局。通过一站式的综合和无忧的平台，它使用户能够快速将AI功能部署到生产环境中，通过单个API轻松访问全面的AI能力。 (网站: [https://edenai.co/](https://edenai.co/))

本示例介绍如何使用LangChain与Eden AI模型进行交互

-----------------------------------------------------------------------------------

`EdenAI`不仅仅是模型调用。它还提供了以下高级功能：

- **多个提供商**：获得各种提供商提供的多样化语言模型，让您可以自由选择最适合您用例的模型。

- **备用机制**：设置备用机制，确保即使主要提供商不可用，您也可以轻松切换到备用提供商以保持无缝运行。

- **使用情况跟踪**：基于项目和API密钥，跟踪使用统计信息。此功能可帮助您有效地监控和管理资源消耗。

- **监控和可观察性**：`EdenAI`在平台上提供全面的监控和可观察性工具。监控语言模型的性能，分析使用模式，并获得有价值的见解，以优化您的应用程序。

访问EDENAI的API需要一个API密钥，

您可以通过创建一个帐户 [https://app.edenai.run/user/register](https://app.edenai.run/user/register) 并转到 [https://app.edenai.run/admin/iam/api-keys](https://app.edenai.run/admin/iam/api-keys) 获取。

一旦我们获得了密钥，我们将希望将其设置为环境变量，方法是运行：

```bash
export EDENAI_API_KEY="..."
```

您可以在API参考文档中找到更多详细信息：[https://docs.edenai.co/reference](https://docs.edenai.co/reference)

如果您不想设置环境变量，您可以直接通过edenai_api_key命名参数传递密钥

在初始化EdenAI Chat Model类时。

```python
from langchain_community.chat_models.edenai import ChatEdenAI
from langchain_core.messages import HumanMessage
```

```python
chat = ChatEdenAI(
    edenai_api_key="...", provider="openai", temperature=0.2, max_tokens=250
)
```

```python
messages = [HumanMessage(content="Hello !")]
chat.invoke(messages)
```

```output
AIMessage(content='Hello! How can I assist you today?')
```

```python
await chat.ainvoke(messages)
```

```output
AIMessage(content='Hello! How can I assist you today?')
```

## 流式处理和批处理

`ChatEdenAI`支持流式处理和批处理。以下是一个示例。

```python
for chunk in chat.stream(messages):
    print(chunk.content, end="", flush=True)
```

```output
Hello! How can I assist you today?
```

```python
chat.batch([messages])
```

```output
[AIMessage(content='Hello! How can I assist you today?')]
```

## 备用机制

使用Eden AI，您可以设置备用机制，以确保即使主要提供商不可用，您也可以轻松切换到备用提供商。

```python
chat = ChatEdenAI(
    edenai_api_key="...",
    provider="openai",
    temperature=0.2,
    max_tokens=250,
    fallback_providers="google",
)
```

在此示例中，如果OpenAI遇到任何问题，您可以使用Google作为备用提供商。

有关Eden AI的更多信息和详细信息，请查看此链接：[https://docs.edenai.co/docs/additional-parameters](https://docs.edenai.co/docs/additional-parameters)

## 链式调用

```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template(
    "What is a good name for a company that makes {product}?"
)
chain = prompt | chat
```

```python
chain.invoke({"product": "healthy snacks"})
```

```output
AIMessage(content='VitalBites')
```