# 如何使用少量示例

:::info 先决条件

本指南假设您熟悉以下概念：

- [提示模板](/docs/concepts/#prompt-templates)

- [示例选择器](/docs/concepts/#example-selectors)

- [LLM](/docs/concepts/#llms)

- [向量存储](/docs/concepts/#vectorstores)

:::

在本指南中，我们将学习如何创建一个简单的提示模板，用于在生成时向模型提供示例输入和输出。向LLM提供少量这样的示例被称为少量示例，这是一种简单但强大的指导生成的方式，在某些情况下可以显著提高模型性能。

少量示例提示模板可以由一组示例或一个负责从定义的集合中选择一部分示例的[示例选择器](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.base.BaseExampleSelector.html)类构建。

本指南将介绍使用字符串提示模板进行少量示例。有关使用聊天消息进行聊天模型的少量示例的指南，请参见[此处](/docs/how_to/few_shot_examples_chat/)。

## 创建少量示例的格式化程序

配置一个格式化程序，将少量示例格式化为字符串。这个格式化程序应该是一个`PromptTemplate`对象。

```python
from langchain_core.prompts import PromptTemplate
example_prompt = PromptTemplate.from_template("问题：{question}\n{answer}")
```

## 创建示例集合

接下来，我们将创建一个少量示例的列表。每个示例应该是一个字典，表示我们上面定义的格式化提示的示例输入。

```python
examples = [
    {
        "question": "谁活得更长，穆罕默德·阿里还是艾伦·图灵？",
        "answer": """
是否需要后续问题：是的。
后续问题：穆罕默德·阿里去世时多大年纪？
中间答案：穆罕默德·阿里去世时74岁。
后续问题：艾伦·图灵去世时多大年纪？
中间答案：艾伦·图灵去世时41岁。
所以最终答案是：穆罕默德·阿里
""",
    },
    {
        "question": "克雷格斯列表的创始人是什么时候出生的？",
        "answer": """
是否需要后续问题：是的。
后续问题：克雷格斯列表的创始人是谁？
中间答案：克雷格斯列表的创始人是克雷格·纽马克。
后续问题：克雷格·纽马克是什么时候出生的？
中间答案：克雷格·纽马克于1952年12月6日出生。
所以最终答案是：1952年12月6日
""",
    },
    {
        "question": "乔治·华盛顿的外祖父是谁？",
        "answer": """
是否需要后续问题：是的。
后续问题：乔治·华盛顿的母亲是谁？
中间答案：乔治·华盛顿的母亲是玛丽·波尔·华盛顿。
后续问题：玛丽·波尔·华盛顿的父亲是谁？
中间答案：玛丽·波尔·华盛顿的父亲是约瑟夫·波尔。
所以最终答案是：约瑟夫·波尔
""",
    },
    {
        "question": "《大白鲨》和《皇家赌场》的导演都来自同一个国家吗？",
        "answer": """
是否需要后续问题：是的。
后续问题：《大白鲨》的导演是谁？
中间答案：《大白鲨》的导演是史蒂文·斯皮尔伯格。
后续问题：史蒂文·斯皮尔伯格来自哪个国家？
中间答案：美国。
后续问题：《皇家赌场》的导演是谁？
中间答案：《皇家赌场》的导演是马丁·坎贝尔。
后续问题：马丁·坎贝尔来自哪个国家？
中间答案：新西兰。
所以最终答案是：不是
""",
    },
]
```

让我们使用其中一个示例测试格式化提示：

```python
print(example_prompt.invoke(examples[0]).to_string())
```

```output
问题：谁活得更长，穆罕默德·阿里还是艾伦·图灵？
是否需要后续问题：是的。
后续问题：穆罕默德·阿里去世时多大年纪？
中间答案：穆罕默德·阿里去世时74岁。
后续问题：艾伦·图灵去世时多大年纪？
中间答案：艾伦·图灵去世时41岁。
所以最终答案是：穆罕默德·阿里
```

### 将示例和格式化程序传递给`FewShotPromptTemplate`

最后，创建一个[`FewShotPromptTemplate`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.few_shot.FewShotPromptTemplate.html)对象。该对象接受少量示例和少量示例的格式化程序。当格式化此`FewShotPromptTemplate`时，它使用`example_prompt`格式化传递的示例，然后将它们添加到`suffix`之前的最终提示中：

```python
from langchain_core.prompts import FewShotPromptTemplate
prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="问题：{input}",
    input_variables=["input"],
)
print(
    prompt.invoke({"input": "乔治·华盛顿的父亲是谁？"}).to_string()
)
```

```output
问题：谁活得更长，穆罕默德·阿里还是艾伦·图灵？
是否需要后续问题：是的。
后续问题：穆罕默德·阿里去世时多大年纪？
中间答案：穆罕默德·阿里去世时74岁。
后续问题：艾伦·图灵去世时多大年纪？
中间答案：艾伦·图灵去世时41岁。
所以最终答案是：穆罕默德·阿里
问题：克雷格斯列表的创始人是什么时候出生的？
是否需要后续问题：是的。
后续问题：克雷格斯列表的创始人是谁？
中间答案：克雷格斯列表的创始人是克雷格·纽马克。
后续问题：克雷格·纽马克是什么时候出生的？
中间答案：克雷格·纽马克于1952年12月6日出生。
所以最终答案是：1952年12月6日
问题：乔治·华盛顿的外祖父是谁？
是否需要后续问题：是的。
后续问题：乔治·华盛顿的母亲是谁？
中间答案：乔治·华盛顿的母亲是玛丽·波尔·华盛顿。
后续问题：玛丽·波尔·华盛顿的父亲是谁？
中间答案：玛丽·波尔·华盛顿的父亲是约瑟夫·波尔。
所以最终答案是：约瑟夫·波尔
问题：《大白鲨》和《皇家赌场》的导演都来自同一个国家吗？
是否需要后续问题：是的。
后续问题：《大白鲨》的导演是谁？
中间答案：《大白鲨》的导演是史蒂文·斯皮尔伯格。
后续问题：史蒂文·斯皮尔伯格来自哪个国家？
中间答案：美国。
后续问题：《皇家赌场》的导演是谁？
中间
```

通过向模型提供这样的示例，我们可以引导模型做出更好的回应。

## 使用示例选择器

我们将重用上一节中的示例集和格式化程序。但是，我们不会直接将示例馈送到 `FewShotPromptTemplate` 对象中，而是将它们馈送到名为 [`SemanticSimilarityExampleSelector`](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.semantic_similarity.SemanticSimilarityExampleSelector.html) 的 `ExampleSelector` 实现实例中。该类根据输入与少样本示例的相似性选择初始集合中的少样本示例。它使用嵌入模型计算输入与少样本示例之间的相似性，以及向量存储库执行最近邻搜索。

为了展示它的样子，让我们初始化一个实例并在隔离环境中调用它：

```python
from langchain_chroma import Chroma
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings
example_selector = SemanticSimilarityExampleSelector.from_examples(
    # 这是可供选择的示例列表。
    examples,
    # 这是用于生成嵌入的嵌入类，用于衡量语义相似性。
    OpenAIEmbeddings(),
    # 这是用于存储嵌入并进行相似性搜索的 VectorStore 类。
    Chroma,
    # 这是要生成的示例数量。
    k=1,
)
# 选择与输入最相似的示例。
question = "Who was the father of Mary Ball Washington?"
selected_examples = example_selector.select_examples({"question": question})
print(f"Examples most similar to the input: {question}")
for example in selected_examples:
    print("\n")
    for k, v in example.items():
        print(f"{k}: {v}")
```

```output
Examples most similar to the input: Who was the father of Mary Ball Washington?
answer: 
Are follow up questions needed here: Yes.
Follow up: Who was the mother of George Washington?
Intermediate answer: The mother of George Washington was Mary Ball Washington.
Follow up: Who was the father of Mary Ball Washington?
Intermediate answer: The father of Mary Ball Washington was Joseph Ball.
So the final answer is: Joseph Ball
question: Who was the maternal grandfather of George Washington?
```

现在，让我们创建一个 `FewShotPromptTemplate` 对象。该对象接受示例选择器和用于少样本示例的格式化程序提示。

```python
prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    suffix="Question: {input}",
    input_variables=["input"],
)
print(
    prompt.invoke({"input": "Who was the father of Mary Ball Washington?"}).to_string()
)
```

```output
Question: Who was the maternal grandfather of George Washington?
Are follow up questions needed here: Yes.
Follow up: Who was the mother of George Washington?
Intermediate answer: The mother of George Washington was Mary Ball Washington.
Follow up: Who was the father of Mary Ball Washington?
Intermediate answer: The father of Mary Ball Washington was Joseph Ball.
So the final answer is: Joseph Ball
Question: Who was the father of Mary Ball Washington?
```

## 下一步

您现在已经学会了如何向您的提示中添加少样本示例。

接下来，请查看本节中有关提示模板的其他操作指南，有关 [与聊天模型进行少样本处理](/docs/how_to/few_shot_examples_chat) 的相关操作指南，或其他 [示例选择器操作指南](/docs/how_to/example_selectors/)。
