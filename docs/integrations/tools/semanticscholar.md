# Semantic Scholar API 工具

这篇笔记展示了如何使用语义学者工具与一个代理进行交互。

```python
# 首先安装语义学者 API
%pip install --upgrade --quiet  semanticscholar
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
```

```python
instructions = """You are an expert researcher."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```

```python
llm = ChatOpenAI(temperature=0)
```

```python
from langchain_community.tools.semanticscholar.tool import SemanticScholarQueryRun
tools = [SemanticScholarQueryRun()]
```

```python
agent = create_openai_functions_agent(llm, tools, prompt)
```

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```

```python
agent_executor.invoke(
    {
        "input": "What are some biases in the large language models? How have people tried to mitigate them? "
        "show me a list of papers and techniques. Based on your findings write new research questions "
        "to work on. Break down the task into subtasks for search. Use the search tool"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
调用: `semanticscholar`，参数为 `{'query': 'biases in large language models'}`
发表年份: 2023
标题: Biases in Large Language Models: Origins, Inventory, and Discussion
作者: Roberto Navigli, Simone Conia, Björn Ross
摘要: 本文介绍和讨论了当前主流自然语言处理方法核心的大型语言模型中存在的偏见问题。我们首先介绍了数据选择偏见，即由构成训练语料库的文本选择引起的偏见。然后，我们调查了在训练在这些语料库上的语言模型生成的文本中表现出的不同类型的社会偏见，从性别到年龄，从性取向到种族，从宗教到文化。最后，我们提出了关于衡量、减少和解决上述类型偏见的方向。
...
> 链结束。
```

根据我的调查结果，以下是一些关于减轻大型语言模型偏见的论文和技术：

1. 《Biases in Large Language Models: Origins, Inventory, and Discussion》由 Roberto Navigli, Simone Conia, Björn Ross：本文讨论了大型语言模型中的偏见问题，包括数据选择偏见和各种社会偏见。它探讨了衡量、减少和解决语言模型中偏见的方向。

2. 《Surfacing Biases in Large Language Models using Contrastive Input Decoding》由 G. Yona, Or Honovich, Itay Laish, Roee Aharoni：本文提出了一种名为 Contrastive Input Decoding (CID) 的解码算法，用于突出语言模型中的特定上下文偏见。它旨在揭示当给定不同输入时模型行为的有意义差异。

3. 《Benchmarking Cognitive Biases in Large Language Models as Evaluators》由 Ryan Koo, Minhwa Lee, Vipul Raheja, Jong Inn Park, Zae Myung Kim, Dongyeop Kang：该工作评估了作为自动评估器使用的大型语言模型中的偏见。它引入了用于衡量模型评估输出中不同认知偏见的认知偏见基准 (CoBBLEr)。

4. 《Should ChatGPT be Biased? Challenges and Risks of Bias in Large Language Models》由 Emilio Ferrara：本文探讨了大规模语言模型中偏见的挑战和风险。它讨论了偏见的起源和偏见模型输出的伦理影响。它还概述了当前用于识别、量化和减轻语言模型中偏见的方法。

5. 《Towards Understanding and Mitigating Social Biases in Language Models》由 P. Liang, Chiyu Wu, Louis-Philippe Morency, R. Salakhutdinov：该工作侧重于减轻语言模型中的社会偏见。它提出了衡量表征性偏见的新基准和指标，并提出了在文本生成过程中减轻偏见的步骤。

6. 《In-Contextual Bias Suppression for Large Language Models》由 Daisuke Oba, Masahiro Kaneko, D. Bollegala：本文提出了一种名为偏见抑制的新方法，用于减轻语言模型中的性别偏见。它使用基于文本的序言和描述性句子来抑制偏见，而无需访问模型参数。

基于这些论文，以下是一些可以研究的新问题：

1. 如何进一步提高大型语言模型中偏见抑制技术的有效性？

2. 语言模型中偏见对下游应用和用户体验的长期影响是什么？

3. 如何开发更全面的基准和指标来衡量和评估语言模型中的偏见？

4. 减轻语言模型中偏见涉及哪些伦理考虑和权衡？

5. 如何确保部署语言模型的透明度和问责制，以最小化偏见？

6. 在特定领域或数据集上微调语言模型引入了哪些潜在偏见，我们如何解决？

为了将任务细分为进一步搜索的子任务，您可以专注于以下主题：

1. 测量和量化大型语言模型中的偏见的技术。

2. 在语言模型文本生成过程中减轻偏见的方法。

3. 评估方法和基准，用于评估语言模型中的偏见。

4. 语言模型中偏见的伦理考虑和影响。

5. 语言模型中偏见对下游应用和用户体验的影响。

6. 不需要访问模型参数的偏见抑制技术。

使用搜索工具，您可以探索这些子主题，并找到更多与每个子任务相关的具体论文和技术。

根据我的调研，以下是一些与减轻大型语言模型中偏见相关的论文和技术：

1. "Biases in Large Language Models: Origins, Inventory, and Discussion"，作者：Roberto Navigli, Simone Conia, Björn Ross：该论文讨论了大型语言模型中的偏见问题，包括数据选择偏见和各种社会偏见。它探讨了衡量、减少和解决语言模型偏见的方向。

2. "Surfacing Biases in Large Language Models using Contrastive Input Decoding"，作者：G. Yona, Or Honovich, Itay Laish, Roee Aharoni：该论文提出了一种称为对比输入解码（CID）的解码算法，以突出语言模型中特定上下文的偏见。它旨在揭示在给定不同输入时模型行为的有意义差异。

3. "Benchmarking Cognitive Biases in Large Language Models as Evaluators"，作者：Ryan Koo, Minhwa Lee, Vipul Raheja, Jong Inn Park, Zae Myung Kim, Dongyeop Kang：该研究评估了作为自动评估器使用的大型语言模型中的偏见。它引入了用于衡量模型评估输出中不同认知偏见的 LLM 作为评估器的认知偏见基准（CoBBLEr）。

4. "Should ChatGPT be Biased? Challenges and Risks of Bias in Large Language Models"，作者：Emilio Ferrara：该论文探讨了大规模语言模型中偏见的挑战和风险。它讨论了偏见的起源以及偏见模型输出的伦理影响。它还概述了当前用于识别、量化和减轻语言模型偏见的方法。

5. "Towards Understanding and Mitigating Social Biases in Language Models"，作者：P. Liang, Chiyu Wu, Louis-Philippe Morency, R. Salakhutdinov：该研究侧重于减轻语言模型中的社会偏见。它提出了衡量表征偏见的新基准和度量标准，并建议在文本生成过程中减轻偏见的步骤。

6. "In-Contextual Bias Suppression for Large Language Models"，作者：Daisuke Oba, Masahiro Kaneko, D. Bollegala：该论文提出了一种称为偏见抑制的新方法，以减轻语言模型中的性别偏见。它使用基于文本的序言和描述性句子来抑制偏见，而无需访问模型参数。

基于这些论文，以下是一些可以研究的新问题：

1. 如何进一步提高大型语言模型中偏见抑制技术的有效性？

2. 语言模型中偏见对下游应用和用户体验的长期影响是什么？

3. 如何开发更全面的基准和度量标准来衡量和评估语言模型中的偏见？

4. 减轻语言模型中偏见涉及哪些伦理考虑和权衡？

5. 如何确保部署语言模型的透明度和问责，以最小化偏见？

6. 在特定领域或数据集上微调语言模型可能引入哪些潜在偏见，以及如何解决？

要将任务细分为进一步搜索的子任务，您可以关注以下主题：

1. 衡量和量化大型语言模型中偏见的技术。

2. 在语言模型的文本生成过程中减轻偏见的方法。

3. 评估方法和基准，用于评估语言模型中的偏见。

4. 语言模型中偏见的伦理考虑和影响。

5. 语言模型中偏见对下游应用和用户体验的影响。

6. 不需要访问模型参数的偏见抑制技术。

使用搜索工具，您可以探索这些子主题，并找到与每个子任务相关的更具体的论文和技术。