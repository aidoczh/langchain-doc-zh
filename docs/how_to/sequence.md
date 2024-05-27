---
sidebar_position: 0
keywords: [Runnable, Runnables, LCEL]
---
# 如何链接可运行对象
:::info 先决条件
本指南假设您熟悉以下概念：
- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [提示模板](/docs/concepts/#prompt-templates)
- [聊天模型](/docs/concepts/#chat-models)
- [输出解析器](/docs/concepts/#output-parsers)
:::
关于[LangChain 表达语言](/docs/concepts/#langchain-expression-language)的一个要点是，任何两个可运行对象都可以被“链接”成序列。前一个可运行对象的`.invoke()`调用的输出将作为输入传递给下一个可运行对象。这可以使用管道运算符 (`|`) 或更明确的`.pipe()`方法来实现，它们的作用是相同的。
生成的[`RunnableSequence`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSequence.html)本身也是一个可运行对象，这意味着它可以被调用、流式处理或者像任何其他可运行对象一样进一步链接。通过这种方式链接可运行对象的优势在于高效的流式处理（一旦序列可用，它就会立即流式处理输出），以及使用像[LangSmith](/docs/how_to/debugging)这样的工具进行调试和跟踪。
## 管道运算符：`|`
为了展示这是如何工作的，让我们通过一个示例来详细介绍。我们将演示 LangChain 中的一个常见模式：使用[提示模板](/docs/how_to#prompt-templates)将输入格式化为[聊天模型](/docs/how_to#chat-models)，最后使用[输出解析器](/docs/how_to#output-parsers)将聊天消息输出转换为字符串。
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")
chain = prompt | model | StrOutputParser()
```
提示和模型都是可运行对象，而提示调用的输出类型与聊天模型的输入类型相同，因此我们可以将它们链接在一起。然后，我们可以像任何其他可运行对象一样调用生成的序列：
```python
chain.invoke({"topic": "bears"})
```
```output
"Here's a bear joke for you:\n\nWhy did the bear dissolve in water?\nBecause it was a polar bear!"
```
### 强制转换
我们甚至可以将这个链与更多的可运行对象组合在一起，创建另一个链。这可能涉及使用其他类型的可运行对象进行一些输入/输出格式化，具体取决于链组件所需的输入和输出。
例如，假设我们想要将生成笑话的链与另一个链组合，用于评估生成的笑话是否好笑。我们需要注意如何将输入格式化为下一个链。在下面的示例中，链中的字典会自动解析并转换为[`RunnableParallel`](/docs/how_to/parallel)，它会并行运行所有值，并返回带有结果的字典。
下面是它的运行方式：
```python
from langchain_core.output_parsers import StrOutputParser
analysis_prompt = ChatPromptTemplate.from_template("is this a funny joke? {joke}")
composed_chain = {"joke": chain} | analysis_prompt | model | StrOutputParser()
composed_chain.invoke({"topic": "bears"})
```
```output
'Haha, that\'s a clever play on words! Using "polar" to imply the bear dissolved or became polar/polarized when put in water. Not the most hilarious joke ever, but it has a cute, groan-worthy pun that makes it mildly amusing. I appreciate a good pun or wordplay joke.'
```
函数也会被强制转换为可运行对象，因此您也可以向您的链中添加自定义逻辑。下面的链结果与之前的逻辑流相同：
```python
composed_chain_with_lambda = (
    chain
    | (lambda input: {"joke": input})
    | analysis_prompt
    | model
    | StrOutputParser()
)
composed_chain_with_lambda.invoke({"topic": "beets"})
```
```output
"Haha, that's a cute and punny joke! I like how it plays on the idea of beets blushing or turning red like someone blushing. Food puns can be quite amusing. While not a total knee-slapper, it's a light-hearted, groan-worthy dad joke that would make me chuckle and shake my head. Simple vegetable humor!"
```
但是，请注意，使用这种函数可能会干扰流式处理等操作。有关更多信息，请参阅[此部分](/docs/how_to/functions)。
## `.pipe()` 方法
我们还可以使用`.pipe()`方法组合相同的序列。具体如下：
```python
from langchain_core.runnables import RunnableParallel
composed_chain_with_pipe = (
    RunnableParallel({"joke": chain})
    .pipe(analysis_prompt)
    .pipe(model)
    .pipe(StrOutputParser())
)
composed_chain_with_pipe.invoke({"topic": "battlestar galactica"})
```
"我无法逐字复制任何受版权保护的材料，但我可以尝试分析您提供的笑话中的幽默，而不是直接引用它。\n\n这个笑话是在于暗示 Cylon 袭击者，他们是《星际大战》宇宙中的反派，在袭击人类母星（十二殖民地）后未能找到幸存者，是因为他们的瞄准系统使用了过时且性能不佳的操作系统（Windows Vista）。\n\n这个笑话的幽默感源于将未来科幻背景与现实世界中令人烦恼的问题相对比——即使用有 bug、慢或不可靠的软件或技术。它取笑了 Windows Vista 的被认为存在的不足之处，当时该系统因性能问题和其他方面的问题而受到广泛批评。\n\n通过将 Cylon 未能找到人类归因于他们使用 Vista，这个笑话在虚构的先进机器人种族和现实世界许多人经历的熟悉技术烦恼之间创造了一个有趣且意想不到的联系。\n\n总的来说，这个笑话依赖于不协调和可关联性来产生幽默，但没有直接复制任何受版权保护的材料。"
```python
composed_chain_with_pipe = RunnableParallel({"joke": chain}).pipe(
    analysis_prompt, model, StrOutputParser()
)
```
## 相关
- [流处理](/docs/how_to/streaming/)：查看流处理指南，了解链的流处理行为
- 
```