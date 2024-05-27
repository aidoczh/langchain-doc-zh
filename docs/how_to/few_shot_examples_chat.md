# 如何在聊天模型中使用少量示例

:::info 前提条件

本指南假定您熟悉以下概念：

- [提示模板](/docs/concepts/#prompt-templates)

- [示例选择器](/docs/concepts/#example-selectors)

- [聊天模型](/docs/concepts/#chat-model)

- [向量存储](/docs/concepts/#vectorstores)

:::

本指南介绍了如何使用示例输入和输出提示聊天模型。向模型提供少量示例被称为少量示例，这是一种简单但强大的引导生成的方式，在某些情况下可以显著改善模型性能。

目前并没有就如何最好地进行少量示例提示达成一致意见，最佳提示编制可能会因模型而异。因此，我们提供了少量示例提示模板，比如[FewShotChatMessagePromptTemplate](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.few_shot.FewShotChatMessagePromptTemplate.html?highlight=fewshot#langchain_core.prompts.few_shot.FewShotChatMessagePromptTemplate)，作为灵活的起点，您可以根据需要修改或替换它们。

少量示例提示模板的目标是根据输入动态选择示例，然后将示例格式化为最终提示，以供模型使用。

**注意：** 以下代码示例仅适用于聊天模型，因为`FewShotChatMessagePromptTemplates`旨在输出格式化的[聊天消息](/docs/concepts/#message-types)，而不是纯字符串。对于与完成模型（LLMs）兼容的纯字符串模板的类似少量示例提示示例，请参阅[少量示例提示模板](/docs/how_to/few_shot_examples/)指南。

## 固定示例

最基本（也是常见）的少量示例提示技术是使用固定提示示例。这样，您可以选择一个链，评估它，并避免在生产中担心额外的移动部件。

模板的基本组件包括：

- `examples`：要包含在最终提示中的字典示例列表。

- `example_prompt`：通过其[`format_messages`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html?highlight=format_messages#langchain_core.prompts.chat.ChatPromptTemplate.format_messages)方法将每个示例转换为1个或多个消息。一个常见的示例是将每个示例转换为一个人类消息和一个AI消息响应，或者是一个人类消息后跟一个函数调用消息。

以下是一个简单的演示。首先，定义您想要包含的示例：

```python
%pip install -qU langchain langchain-openai langchain-chroma
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

```output
警告：您正在使用 pip 版本 22.0.4；然而，版本 24.0 可用。
您应该考虑通过'/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip'命令进行升级。
注意：您可能需要重新启动内核以使用更新后的软件包。
```

```python
from langchain_core.prompts import ChatPromptTemplate, FewShotChatMessagePromptTemplate
examples = [
    {"input": "2+2", "output": "4"},
    {"input": "2+3", "output": "5"},
]
```

接下来，将它们组装成少量示例提示模板。

```python
# 这是用于格式化每个单独示例的提示模板。
example_prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}"),
        ("ai", "{output}"),
    ]
)
few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)
print(few_shot_prompt.invoke({}).to_messages())
```

```output
[HumanMessage(content='2+2'), AIMessage(content='4'), HumanMessage(content='2+3'), AIMessage(content='5')]
```

最后，我们按照下面所示组装最终提示，直接将`few_shot_prompt`传递到`from_messages`工厂方法中，并将其与模型一起使用：

```python
final_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a wondrous wizard of math."),
        few_shot_prompt,
        ("human", "{input}"),
    ]
)
```

```python
from langchain_openai import ChatOpenAI
chain = final_prompt | ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0.0)
chain.invoke({"input": "What's the square of a triangle?"})
```

```output
AIMessage(content='A triangle does not have a square. The square of a number is the result of multiplying the number by itself.', response_metadata={'token_usage': {'completion_tokens': 23, 'prompt_tokens': 52, 'total_tokens': 75}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-3456c4ef-7b4d-4adb-9e02-8079de82a47a-0')
```

## 动态少量示例提示

有时候，你可能只想从整个集合中选择几个示例来展示，这取决于输入。为此，你可以将传递给 `FewShotChatMessagePromptTemplate` 的 `examples` 替换为一个 `example_selector`。其他组件与上面的相同！我们的动态 few-shot prompt 模板如下所示：

- `example_selector`：负责选择给定输入的 few-shot 示例（以及它们返回的顺序）。它们实现了 [BaseExampleSelector](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.base.BaseExampleSelector.html?highlight=baseexampleselector#langchain_core.example_selectors.base.BaseExampleSelector) 接口。一个常见的例子是基于 vectorstore 的 [SemanticSimilarityExampleSelector](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.semantic_similarity.SemanticSimilarityExampleSelector.html?highlight=semanticsimilarityexampleselector#langchain_core.example_selectors.semantic_similarity.SemanticSimilarityExampleSelector)

- `example_prompt`：通过其 [`format_messages`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html?highlight=chatprompttemplate#langchain_core.prompts.chat.ChatPromptTemplate.format_messages) 方法将每个示例转换为一个或多个消息。一个常见的例子是将每个示例转换为一个人类消息和一个 AI 消息响应，或者一个人类消息后跟一个函数调用消息。

这些组件可以与其他消息和聊天模板组合在一起，组装成最终的 prompt。

让我们通过 `SemanticSimilarityExampleSelector` 来举个例子。由于这个实现使用 vectorstore 来根据语义相似性选择示例，我们首先需要填充 vectorstore。基本思想是我们希望搜索并返回与文本输入最相似的示例，因此我们嵌入了 prompt 示例的 `values` 而不是考虑键：

```python
from langchain_chroma import Chroma
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings
examples = [
    {"input": "2+2", "output": "4"},
    {"input": "2+3", "output": "5"},
    {"input": "2+4", "output": "6"},
    {"input": "What did the cow say to the moon?", "output": "nothing at all"},
    {"input": "Write me a poem about the moon", "output": "One for the moon, and one for me, who are we to talk about the moon?"},
]
to_vectorize = [" ".join(example.values()) for example in examples]
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_texts(to_vectorize, embeddings, metadatas=examples)
```

### 创建 `example_selector`

有了创建好的 vectorstore，我们可以创建 `example_selector`。在这里，我们将单独调用它，并将 `k` 设置为只获取两个最接近输入的示例。

```python
example_selector = SemanticSimilarityExampleSelector(
    vectorstore=vectorstore,
    k=2,
)
# prompt 模板将通过将输入传递给 `select_examples` 方法来加载示例
example_selector.select_examples({"input": "horse"})
```

```output
[{'input': 'What did the cow say to the moon?', 'output': 'nothing at all'},
 {'input': '2+4', 'output': '6'}]
```

### 创建 prompt 模板

现在，我们使用上面创建的 `example_selector` 组装 prompt 模板。

```python
from langchain_core.prompts import ChatPromptTemplate, FewShotChatMessagePromptTemplate
# 定义 few-shot prompt。
few_shot_prompt = FewShotChatMessagePromptTemplate(
    # 输入变量选择要传递给 example_selector 的值
    input_variables=["input"],
    example_selector=example_selector,
    # 定义每个示例的格式。
    # 在这种情况下，每个示例将变成 2 条消息：
    # 1 条人类消息和 1 条 AI 消息
    example_prompt=ChatPromptTemplate.from_messages(
        [("human", "{input}"), ("ai", "{output}")]
    ),
)
print(few_shot_prompt.invoke(input="What's 3+3?").to_messages())
```

```output
[HumanMessage(content='2+3'), AIMessage(content='5'), HumanMessage(content='2+2'), AIMessage(content='4')]
```

然后，我们可以将这个 few-shot chat message prompt 模板传递给另一个 chat prompt 模板：

```python
final_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a wondrous wizard of math."),
        few_shot_prompt,
        ("human", "{input}"),
    ]
)
print(few_shot_prompt.invoke(input="What's 3+3?"))
```

```output
messages=[HumanMessage(content='2+3'), AIMessage(content='5'), HumanMessage(content='2+2'), AIMessage(content='4')]
```

### 与 chat model 一起使用

最后，你可以将你的模型连接到 few-shot prompt。

```python
chain = final_prompt | ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0.0)
chain.invoke({"input": "What's 3+3?"})
```

## 下一步

你已经学会了如何在聊天提示中添加少量示例。

接下来，请查看本节中有关提示模板的其他指南，以及有关[使用文本补全模型进行少量示例的指南](/docs/how_to/few_shot_examples)或其他[示例选择器指南](/docs/how_to/example_selectors/)。