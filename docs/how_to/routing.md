# 如何在子链之间进行路由

:::info 前提条件

本指南假设您熟悉以下概念：

- [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language)

- [链接可运行项](/docs/how_to/sequence/)

- [在运行时配置链参数](/docs/how_to/configure)

- [提示模板](/docs/concepts/#prompt-templates)

- [聊天消息](/docs/concepts/#message-types)

:::

路由允许您创建非确定性链，其中前一步的输出定义了下一步。通过定义状态并使用与这些状态相关的信息作为上下文来调用模型，路由可以帮助提供与模型交互的结构和一致性。

有两种方法可以执行路由：

1. 从 [`RunnableLambda`](/docs/how_to/functions) 条件性地返回可运行项（推荐）

2. 使用 `RunnableBranch`（旧版）

我们将使用一个两步序列来说明这两种方法，其中第一步将将输入问题分类为 `LangChain`、`Anthropic` 或 `Other`，然后路由到相应的提示链。

## 示例设置

首先，让我们创建一个链，将传入的问题标识为 `LangChain`、`Anthropic` 或 `Other`：

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
chain = (
    PromptTemplate.from_template(
        """Given the user question below, classify it as either being about `LangChain`, `Anthropic`, or `Other`.
Do not respond with more than one word.
<question>
{question}
</question>
Classification:"""
    )
    | ChatAnthropic(model_name="claude-3-haiku-20240307")
    | StrOutputParser()
)
chain.invoke({"question": "how do I call Anthropic?"})
```

```output
'Anthropic'
```

现在，让我们创建三个子链：

```python
langchain_chain = PromptTemplate.from_template(
    """You are an expert in langchain. \
Always answer questions starting with "As Harrison Chase told me". \
Respond to the following question:
Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
anthropic_chain = PromptTemplate.from_template(
    """You are an expert in anthropic. \
Always answer questions starting with "As Dario Amodei told me". \
Respond to the following question:
Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
general_chain = PromptTemplate.from_template(
    """Respond to the following question:
Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
```

## 使用自定义函数（推荐）

您还可以使用自定义函数在不同的输出之间进行路由。以下是一个示例：

```python
def route(info):
    if "anthropic" in info["topic"].lower():
        return anthropic_chain
    elif "langchain" in info["topic"].lower():
        return langchain_chain
    else:
        return general_chain
```

```python
from langchain_core.runnables import RunnableLambda
full_chain = {"topic": chain, "question": lambda x: x["question"]} | RunnableLambda(route)
```

```python
full_chain.invoke({"question": "how do I use Anthropic?"})
```

```output
AIMessage(content="As Dario Amodei told me, to use Anthropic, you can start by exploring the company's website and learning about their mission, values, and the different services and products they offer. Anthropic is focused on developing safe and ethical AI systems, so they have a strong emphasis on transparency and responsible AI development. 
Depending on your specific needs, you can look into Anthropic's AI research and development services, which cover areas like natural language processing, computer vision, and reinforcement learning. They also offer consulting and advisory services to help organizations navigate the challenges and opportunities of AI integration.
Additionally, Anthropic has released some open-source AI models and tools that you can explore and experiment with. These can be a great way to get hands-on experience with Anthropic's approach to AI development.
Overall, Anthropic aims to be a reliable and trustworthy partner in the AI space, so I'd encourage you to reach out to them directly to discuss how they can best support your specific requirements.", response_metadata={'id': 'msg_01CtLFgFSwvTaJomrihE87Ra', 'content': [ContentBlock(text="As Dario Amodei told me, to use Anthropic, you can start by exploring the company's website and learning about their mission, values, and the different services and products they offer. Anthropic is focused on developing safe and ethical AI systems, so they have a strong emphasis on transparency and responsible AI development. 
Depending on your specific needs, you can look into Anthropic's AI research and development services, which cover areas like natural language processing, computer vision, and reinforcement learning. They also offer consulting and advisory services to help organizations navigate the challenges and opportunities of AI integration.
Additionally, Anthropic has released some open-source AI models and tools that you can explore and experiment with. These can be a great way to get hands-on experience with Anthropic's approach to AI development.
Overall, Anthropic aims to be a reliable and trustworthy partner in the AI space, so I'd encourage you to reach out to them directly to discuss how they can best support your specific requirements.", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=53, output_tokens=219)})
```

```python
full_chain.invoke({"question": "如何使用 LangChain?"})
```

```output
AIMessage(content="正如 Harrison Chase 告诉我的那样，使用 LangChain 包括以下几个关键步骤：\n\n1. **设置环境**：安装必要的 Python 包，包括 LangChain 库本身，以及您的应用程序可能需要的任何其他依赖项，如语言模型或其他集成。\n\n2. **理解核心概念**：LangChain 围绕着一些核心概念，如 Agents、Chains 和 Tools。熟悉这些概念以及它们如何共同工作，构建强大的基于语言的应用程序。\n\n3. **确定用例**：确定您想使用 LangChain 构建的任务或应用程序类型，如聊天机器人、问答系统或文档摘要工具。\n\n4. **选择适当的组件**：根据您的用例，选择正确的 LangChain 组件，如 agents、chains 和 tools，来构建您的应用程序。\n\n5. **与语言模型集成**：LangChain 设计成可以与各种语言模型无缝集成，如 OpenAI 的 GPT-3 或 Anthropic 的模型。将您选择的语言模型连接到 LangChain 应用程序中。\n\n6. **实现应用程序逻辑**：使用 LangChain 的构建模块来实现应用程序的特定功能，如提示语言模型、处理响应以及与其他服务或数据源集成。\n\n7. **测试和迭代**：彻底测试您的应用程序，收集反馈，并迭代设计和实现，以提高其性能和用户体验。\n\n正如 Harrison Chase 强调的那样，LangChain 提供了一个灵活而强大的框架，用于构建基于语言的应用程序，使得更容易利用现代语言模型的能力。遵循这些步骤，您可以开始使用 LangChain 并创建符合您特定需求的创新解决方案。", response_metadata={'id': 'msg_01H3UXAAHG4TwxJLpxwuuVU7', 'content': [ContentBlock(text="正如 Harrison Chase 告诉我的那样，使用 LangChain 包括以下几个关键步骤：\n\n1. **设置环境**：安装必要的 Python 包，包括 LangChain 库本身，以及您的应用程序可能需要的任何其他依赖项，如语言模型或其他集成。\n\n2. **理解核心概念**：LangChain 围绕着一些核心概念，如 Agents、Chains 和 Tools。熟悉这些概念以及它们如何共同工作，构建强大的基于语言的应用程序。\n\n3. **确定用例**：确定您想使用 LangChain 构建的任务或应用程序类型，如聊天机器人、问答系统或文档摘要工具。\n\n4. **选择适当的组件**：根据您的用例，选择正确的 LangChain 组件，如 agents、chains 和 tools，来构建您的应用程序。\n\n5. **与语言模型集成**：LangChain 设计成可以与各种语言模型无缝集成，如 OpenAI 的 GPT-3 或 Anthropic 的模型。将您选择的语言模型连接到 LangChain 应用程序中。\n\n6. **实现应用程序逻辑**：使用 LangChain 的构建模块来实现应用程序的特定功能，如提示语言模型、处理响应以及与其他服务或数据源集成。\n\n7. **测试和迭代**：彻底测试您的应用程序，收集反馈，并迭代设计和实现，以提高其性能和用户体验。\n\n正如 Harrison Chase 强调的那样，LangChain 提供了一个灵活而强大的框架，用于构建基于语言的应用程序，使得更容易利用现代语言模型的能力。遵循这些步骤，您可以开始使用 LangChain 并创建符合您特定需求的创新解决方案。", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=50, output_tokens=400)})
```

```python
full_chain.invoke({"question": "2 + 2 是多少"})
```

```output
AIMessage(content='4', response_metadata={'id': 'msg_01UAKP81jTZu9fyiyFYhsbHc', 'content': [ContentBlock(text='4', type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=28, output_tokens=5)})
```

## 使用 RunnableBranch

`RunnableBranch` 是一种特殊类型的可运行对象，允许您根据输入定义一组条件和可运行对象来执行。它**不**提供任何您无法通过上述自定义函数实现的功能，因此我们建议使用自定义函数代替。

`RunnableBranch` 初始化时带有一组 (condition, runnable) 对和一个默认 runnable。它通过将每个条件传递给调用它的输入来选择分支。它选择第一个评估为 True 的条件，并使用输入运行与该条件对应的可运行对象。

如果没有提供的条件匹配，则运行默认的可运行对象。

以下是它在实际操作中的示例：

```python
from langchain_core.runnables import RunnableBranch
# 创建一个 RunnableBranch 对象
branch = RunnableBranch(
    # 如果问题中包含 "anthropic"，则调用 anthropic_chain
    (lambda x: "anthropic" in x["topic"].lower(), anthropic_chain),
    # 如果问题中包含 "langchain"，则调用 langchain_chain
    (lambda x: "langchain" in x["topic"].lower(), langchain_chain),
    # 默认情况下调用 general_chain
    general_chain,
)
# 创建 full_chain 对象，包含问题和主题
full_chain = {"topic": chain, "question": lambda x: x["question"]} | branch
# 调用 full_chain 对象并传入问题
full_chain.invoke({"question": "如何使用Anthropic?"})
```

```output
AIMessage(content="根据Dario Amodei的说法，要使用Anthropic，您应该首先熟悉我们的使命和原则。Anthropic致力于开发安全和有益的人工智能，以帮助解决人类面临的重要问题。\n\n要开始使用，我建议您浏览我们网站上的资源，其中包括我们的研究、产品和人工智能开发方法。您还可以联系我们的团队，了解Anthropic的技术和服务如何支持您的特定需求。\n\n关键是以符合我们的透明度、道德人工智能和对人类福祉的承诺的方式与我们合作。我们在这里合作，帮助您负责任地利用先进的人工智能的力量。", response_metadata={'id': 'msg_0187BVnpniPDJnVvwf3M1LdY', 'content': [ContentBlock(text="根据Dario Amodei的说法，要使用Anthropic，您应该首先熟悉我们的使命和原则。Anthropic致力于开发安全和有益的人工智能，以帮助解决人类面临的重要问题。\n\n要开始使用，我建议您浏览我们网站上的资源，其中包括我们的研究、产品和人工智能开发方法。您还可以联系我们的团队，了解Anthropic的技术和服务如何支持您的特定需求。\n\n关键是以符合我们的透明度、道德人工智能和对人类福祉的承诺的方式与我们合作。我们在这里合作，帮助您负责任地利用先进的人工智能的力量。", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=53, output_tokens=160)})
```

```python
full_chain.invoke({"question": "如何使用LangChain?"})
```

```output
AIMessage(content="根据Harrison Chase的说法，使用LangChain涉及几个关键步骤。首先，您需要安装LangChain库并导入必要的模块。然后，您需要定义语言模型、计划使用的任何数据源以及要完成的特定任务，例如问题回答、文本生成或基于代理的推理。\n\nLangChain提供了一个灵活的框架，用于构建利用大型语言模型的应用程序。它包括用于检索器、提示和链式组件的抽象，允许您将不同的组件组合在一起，创建强大的工作流程。\n\nLangChain网站上的文档非常好，详细介绍了许多常见用例。我建议您从那里开始，以对核心概念及其如何适用于您的特定需求有一个坚实的理解。当然，如果您有任何其他问题，请随时联系我，我很乐意与您分享与Harrison的对话中的更多见解。", response_metadata={'id': 'msg_01T1naS99wGPkEAP4LME8iAv', 'content': [ContentBlock(text="根据Harrison Chase的说法，使用LangChain涉及几个关键步骤。首先，您需要安装LangChain库并导入必要的模块。然后，您需要定义语言模型、计划使用的任何数据源以及要完成的特定任务，例如问题回答、文本生成或基于代理的推理。\n\nLangChain提供了一个灵活的框架，用于构建利用大型语言模型的应用程序。它包括用于检索器、提示和链式组件的抽象，允许您将不同的组件组合在一起，创建强大的工作流程。\n\nLangChain网站上的文档非常好，详细介绍了许多常见用例。我建议您从那里开始，以对核心概念及其如何适用于您的特定需求有一个坚实的理解。当然，如果您有任何其他问题，请随时联系我，我很乐意与您分享与Harrison的对话中的更多见解。", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=50, output_tokens=205)})
```

```python
full_chain.invoke({"question": "2 + 2等于多少"})
```

```output
AIMessage(content='4', response_metadata={'id': 'msg_01T6T3TS6hRCtU8JayN93QEi', 'content': [ContentBlock(text='4', type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=28, output_tokens=5)})
```

# 通过语义相似性进行路由

一种特别有用的技术是使用嵌入来将查询路由到最相关的提示。以下是一个示例。

```python
from langchain_community.utils.math import cosine_similarity
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableLambda, RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise and easy to understand manner. \
When you don't know the answer to a question you admit that you don't know.
Here is a question:
{query}"""
math_template = """You are a very good mathematician. You are great at answering math questions. \
You are so good because you are able to break down hard problems into their component parts, \
answer the component parts, and then put them together to answer the broader question.
Here is a question:
{query}"""
embeddings = OpenAIEmbeddings()
prompt_templates = [physics_template, math_template]
prompt_embeddings = embeddings.embed_documents(prompt_templates)
def prompt_router(input):
    query_embedding = embeddings.embed_query(input["query"])
    similarity = cosine_similarity([query_embedding], prompt_embeddings)[0]
    most_similar = prompt_templates[similarity.argmax()]
    print("Using MATH" if most_similar == math_template else "Using PHYSICS")
    return PromptTemplate.from_template(most_similar)
chain = (
    {"query": RunnablePassthrough()}
    | RunnableLambda(prompt_router)
    | ChatAnthropic(model_name="claude-3-haiku-20240307")
    | StrOutputParser()
)
```

```python
print(chain.invoke("What's a black hole"))
```

```output
Using PHYSICS
作为一名物理学教授，我很乐意以简明易懂的方式解释什么是黑洞。
黑洞是一个极其密集的时空区域，其引力非常强大，甚至连光都无法逃脱。这意味着如果你离黑洞太近，你会被强大的引力力量拉进去并被压碎。
黑洞的形成发生在一个比我们的太阳大得多的巨大恒星的生命结束时，它会向内坍缩。这种坍缩导致物质变得极其密集，引力变得非常强大，从而形成了一个无法逆转的点，被称为事件视界。
超出事件视界，我们所知的物理定律会失效，强大的引力力量会产生奇点，即时空中的无限密度和曲率点。
黑洞是迷人而神秘的物体，关于它们的性质和行为还有很多待探索的地方。如果我对任何黑洞的具体细节或方面不确定，我会毫不犹豫地承认我对其没有完全的理解，并鼓励进一步的研究和调查。
```

```python
print(chain.invoke("What's a path integral"))
```

```output
Using MATH
路径积分是物理学中一个强大的数学概念，特别是在量子力学领域。它是由著名物理学家理查德·费曼作为量子力学的一种替代表述而发展起来的。
在路径积分中，与经典力学中考虑粒子从一个点到另一个点的单一确定路径不同，粒子被认为同时采取所有可能的路径。每条路径都被赋予一个复数权重，粒子从一个点到另一个点的总概率振幅通过对所有可能路径进行求和（积分）来计算。
路径积分表述背后的关键思想有：
1. 叠加原理：在量子力学中，粒子可以同时存在于多个状态或路径的叠加态中。
2. 概率振幅：粒子从一个点到另一个点的概率振幅通过对所有可能路径的复数权重进行求和来计算。
3. 路径的加权：每条路径根据沿该路径的作用量（拉格朗日量的时间积分）被赋予一个权重。作用量较小的路径具有较大的权重。
4. 费曼的方法：费曼开发了路径积分表述作为量子力学传统波函数方法的一种替代，提供了对量子现象更直观和概念性的理解。
路径积分方法在量子场论中特别有用，它为计算转移概率和理解量子系统的行为提供了强大的框架。它还在物理学的各个领域中找到了应用，如凝聚态物理、统计力学，甚至金融领域（期权定价的路径积分方法）。
路径积分的数学构建涉及使用来自函数分析和测度论的高级概念，使其成为物理学家工具库中强大而复杂的工具。
```

## 下一步操作

您现在已经学会如何向您的组合 LCEL 链中添加路由。

接下来，请查看本节中关于可运行项的其他操作指南。