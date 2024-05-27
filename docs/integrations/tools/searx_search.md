

# SearxNG 搜索

本文介绍如何使用自托管的 `SearxNG` 搜索 API 来搜索网络。

您可以查看[此链接](https://docs.searxng.org/dev/search_api.html)以获取有关 `Searx API` 参数的更多信息。

```python
import pprint
from langchain_community.utilities import SearxSearchWrapper
```
```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888")
```

对于某些搜索引擎，如果有直接的 `answer` 可用，包装器将打印答案而不是完整的搜索结果列表。如果您想获取所有结果，可以使用包装器的 `results` 方法。

```python
search.run("法国的首都是什么")
```
```output
'巴黎是法国的首都，是欧洲最大的国家，面积为 550,000 平方公里（拥有 6500 万居民）。截至 2011 年底，巴黎有 2,234 万居民。她是法兰西岛地区（1200 万人口）的核心。'
```

## 自定义参数

SearxNG 支持[135个搜索引擎](https://docs.searxng.org/user/configured_engines.html)。您还可以使用任意命名的参数自定义 Searx 包装器，这些参数将传递给 Searx 搜索 API。在下面的示例中，我们将更有趣地使用来自 Searx 搜索 API 的自定义搜索参数。

在此示例中，我们将使用 `engines` 参数来查询维基百科。

```python
search = SearxSearchWrapper(
    searx_host="http://127.0.0.1:8888", k=5
)  # k 代表最大项目数
```
```python
search.run("大型语言模型", engines=["wiki"])
```
```output
'大型语言模型（LLMs）代表人工智能的重大进步，承诺通过学习的知识改变领域。过去几年，LLM 的规模每年增长 10 倍，随着这些模型的复杂性和规模增长，它们的功能也在增强。
GPT-3 能够翻译语言、撰写文章、生成计算机代码等，而且几乎不需要监督。2020 年 7 月，OpenAI 推出了 GPT-3，这是当时已知的最大语言模型。简单来说，GPT-3 被训练用于预测句子中的下一个单词，就像文本消息自动完成功能的工作原理。
大型语言模型（LLM）是一种深度学习算法，可以识别、总结、翻译、预测和生成基于大规模数据集知识的文本和其他内容。大型语言模型是变压器模型最成功的应用之一。
当今众所周知的语言模型，例如 OpenAI 的 GPT-3、Google 的 PaLM 或 LaMDA、Meta 的 Galactica 或 OPT、Nvidia/Microsoft 的 Megatron-Turing、AI21 Labs 的 Jurassic-1 等...
大型语言模型（LLMs）如 GPT-3 越来越多地被用于生成文本。这些工具应谨慎使用，因为它们可能生成具有偏见、不可验证、构成原创研究或侵犯版权的内容。'
```

传递其他 Searx 参数给 Searx，如 `language`。

```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888", k=1)
search.run("深度学习", language="es", engines=["wiki"])
```
```output
'深度学习（Deep Learning）是一组机器学习算法，试图使用支持数据的多次非线性转换的计算架构来对数据中的高级抽象进行建模。'
```

## 获取带有元数据的结果

在此示例中，我们将使用 `categories` 参数查找科学论文，并将结果限制为 `time_range`（并非所有引擎都支持时间范围选项）。

我们还希望以结构化方式获取结果，包括元数据。为此，我们将使用包装器的 `results` 方法。

```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888")
```
```python
results = search.results(
    "大型语言模型提示",
    num_results=5,
    categories="science",
    time_range="year",
)
pprint.pp(results)
```
```output
[{'snippet': '… 自然语言指令，大型语言模型（… 用于引导模型的提示，以及最有效的提示… 对于提示工程，我们提出了自动提示…',
  'title': '大型语言模型是人类级提示工程师',
  'link': 'https://arxiv.org/abs/2211.01910',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… 大型语言模型（LLMs）为使用 AI 进行原型设计带来了新的可能性 [18]。在大量文本数据的预训练下，模型… 被称为提示的自然语言指令。…',
  'title': 'Promptchainer：通过可视化编程链接大型语言模型提示',
  'link': 'https://dl.acm.org/doi/abs/10.1145/3491101.3519729',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… 可以内省大提示模型。我们从 T01 推导出视图 ϕ0(X) 和模型 h0。然而，在共同训练期间，我们专注于软提示调整，…',
  'title': '共同训练改进大型语言模型的基于提示的学习',
  'link': 'https://proceedings.mlr.press/v162/lang22a.html',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… 随着代码的大型语言模型（LLMs）的成功以及它们作为… 提示设计过程变得重要。在这项工作中，我们提出了一个名为 Repo-Level Prompt…',
  'title': '用于代码大型语言模型的存储库级提示生成',
  'link': 'https://arxiv.org/abs/2206.12839',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… 图 2 | 从最大语言模型（Gopher）的提示不同组件的好处，从分层 logistic 回归的估计中得出。每个点估计唯一…',
  'title': '语言模型能否从上下文中的解释中学习？',
  'link': 'https://arxiv.org/abs/2204.02329',
  'engines': ['google scholar'],
  'category': 'science'}]
```
```python
从arxiv获取论文
```python

results = search.results(

    "大型语言模型提示", num_results=5, engines=["arxiv"]

)

pprint.pp(results)

```
```output

[{'snippet': '由于大型预训练语言模型的先进改进，基于提示的微调在各种下游任务上被证明是有效的。尽管已经调查了许多提示方法，但仍然不清楚三种提示类型（即人工设计的提示、模式提示和空提示）中哪种提示最有效。在这项工作中，我们在少样本和完全监督的情况下对比了三种类型的提示。我们的实验结果表明，模式提示在一般情况下最有效。此外，当训练数据规模变大时，性能差距往往会减小。',

  'title': 'Do Prompts Solve NLP Tasks Using Natural Language?',

  'link': 'http://arxiv.org/abs/2203.00902v1',

  'engines': ['arxiv'],

  'category': 'science'},

 {'snippet': '交叉提示自动化作文评分（AES）要求系统使用非目标提示的文章为目标提示的文章评分。由于获取特定提示的大量预分级文章通常是困难和不现实的，交叉提示AES的任务对于实际AES系统的发展至关重要，但仍然是一个未充分研究的领域。为特定提示AES设计的模型严重依赖于特定提示的知识，在交叉提示设置下表现不佳，而当前的交叉提示AES方法要么需要一定数量的标记目标提示文章，要么需要大量未标记的目标提示文章以多步方式进行迁移学习。为了解决这些问题，我们引入了适用于交叉提示AES的Prompt Agnostic Essay Scorer（PAES）。我们的方法在训练过程中不需要访问标记或未标记的目标提示数据，是一种单阶段方法。PAES在实践中易于应用，并在Automated Student Assessment Prize（ASAP）数据集上实现了最先进的性能。',

  'title': 'Prompt Agnostic Essay Scorer: A Domain Generalization Approach to '

           'Cross-prompt Automated Essay Scoring',

  'link': 'http://arxiv.org/abs/2008.01441v1',

  'engines': ['arxiv'],

  'category': 'science'},

 {'snippet': '提示研究表明，在许多任务中，即使几乎没有监督训练，提示也表现出色。然而，机器翻译的提示仍然在文献中未充分探讨。我们通过对翻译的提示策略进行系统研究，检查提示模板和示例选择的各种因素。我们进一步探讨了单语数据的使用以及跨语言、跨领域和句子到文档的提示学习的可行性。对GLM-130B（Zeng等，2022）的广泛实验表明：1）提示示例的数量和质量很重要，使用次优示例会降低翻译质量；2）提示示例的几个特征，如语义相似性，与其提示性能呈显著的Spearman相关性，但没有一个相关性足够强；3）通过从单语数据构建的伪平行提示示例进行零-shot提示可以改善翻译；4）通过从其他设置中选择的提示示例转移知识可以实现改进的性能。最后，我们对模型输出进行了分析，并讨论了提示仍然存在的几个问题。',

  'title': 'Prompting Large Language Model for Machine Translation: A Case '

           'Study',

  'link': 'http://arxiv.org/abs/2301.07069v2',

  'engines': ['arxiv'],

  'category': 'science'},

 {'snippet': '大型语言模型可以以零-shot方式执行新任务，只需使用指定所需行为的自然语言提示。这些提示通常是手工设计的，但也可以通过梯度方法从标记数据中学习。'

```
然而，目前尚未有人深入探讨有效提示的因素，尤其是当提示是自然语言时。在本文中，我们研究了有效提示所共享的常见属性。我们首先提出了一种基于 Langevin 动力学的人类可读提示调整方法（F LUENT P ROMPT），该方法结合了流畅性约束，以找到有效且流畅的提示的多样分布。我们的分析表明，有效的提示与任务领域有关，并校准标签词的先验概率。基于这些发现，我们还提出了一种仅使用未标记数据生成提示的方法，其在三个任务中平均准确率超过强基线 7.0%。
- 标题：《走向人类可读提示调整：《闪灵》是一部好电影，也是一个好提示吗？》
- 链接：[arXiv](http://arxiv.org/abs/2212.10539v1)
- 类别：科学
---
目前用于将大型生成语言模型映射到监督任务的方法可能无法充分探索模型的新颖能力。以 GPT-3 为案例研究，我们表明零提示可以显著优于少提示。我们认为，在这些情况下，少量示例的功能更好地描述为定位已学习任务，而不是元学习。这一分析促使重新思考提示在控制和评估强大语言模型中的作用。在这项工作中，我们讨论了提示编程的方法，强调通过自然语言的视角考虑提示的有用性。我们探讨了利用叙述和文化锚点的能力来编码微妙意图的技术，以及在产生结论之前鼓励将问题分解为组件的技术。在这更全面的提示编程理论的指导下，我们还介绍了元提示的概念，该提示为模型生成自己的自然语言提示，用于一系列任务。最后，我们讨论了如何将这些更一般的与语言模型交互的方法纳入现有和未来的基准测试和实际应用中。
- 标题：《大型语言模型的提示编程：超越少提示范式》
- 链接：[arXiv](http://arxiv.org/abs/2102.07350v1)
- 类别：科学
---
我们还可以直接查询来自 `github` 和其他源代码库的结果。
```python
results = search.results("large language model", num_results=20, engines=["github", "gitlab"])
pprint.pp(results)
```
```output

[{'snippet': "Implementation of 'A Watermark for Large Language Models' paper by Kirchenbauer & Geiping et. al.",

  'title': 'Peutlefaire / LMWatermark',

  'link': 'https://gitlab.com/BrianPulfer/LMWatermark',

  'engines': ['gitlab'],

  'category': 'it'},

 {'snippet': 'Guide to using pre-trained large language models of source code',

  'title': 'Code-LMs',

  'link': 'https://github.com/VHellendoorn/Code-LMs',

  'engines': ['github'],

  'category': 'it'},

 {'snippet': '',

  'title': 'Simen Burud / Large-scale Language Models for Conversational Speech Recognition',

  'link': 'https://gitlab.com/BrianPulfer',

  'engines': ['gitlab'],

  'category': 'it'},

 {'snippet': 'Dramatron uses large language models to generate coherent scripts and screenplays.',

  'title': 'dramatron',

  'link': 'https://github.com/deepmind/dramatron',

  'engines': ['github'],

  'category': 'it'},

 {'snippet': 'Code for loralib, an implementation of "LoRA: Low-Rank' 

```
以上是对 `it` 类别下的 `large language model` 进行查询的示例。然后我们过滤出来自 github 的结果。
```python
pprint.pp(list(filter(lambda r: r["engines"][0] == "github", results)))
```
```output

[{'snippet': 'Guide to using pre-trained large language models of source code',

  'title': 'Code-LMs',

  'link': 'https://github.com/VHellendoorn/Code-LMs',

  'engines': ['github'],

  'category': 'it'},

 {'snippet': 'Dramatron uses large language models to generate coherent scripts and screenplays.',

  'title': 'dramatron',

  'link': 'https://github.com/deepmind/dramatron',

  'engines': ['github'],

  'category': 'it'}]

```
我们还可以直接查询来自 `github` 和其他源代码库的结果。
```python
results = search.results("large language model", num_results=20, engines=["github", "gitlab"])
pprint.pp(results)
```
```output

[{'snippet': "Implementation of 'A Watermark for Large Language Models' paper by Kirchenbauer & Geiping et. al.",

  'title': 'Peutlefaire / LMWatermark',

  'link': 'https://gitlab.com/BrianPulfer/LMWatermark',

  'engines': ['gitlab'],

  'category': 'it'},

 {'snippet': 'Guide to using pre-trained large language models of source code',

  'title': 'Code-LMs',

  'link': 'https://github.com/VHellendoorn/Code-LMs',

  'engines': ['github'],

  'category': 'it'},

 {'snippet': '',

  'title': 'Simen Burud / Large-scale Language Models for Conversational Speech Recognition',

  'link': 'https://gitlab.com/BrianPulfer',

  'engines': ['gitlab'],

  'category': 'it'},

 {'snippet': 'Dramatron uses large language models to generate coherent scripts and screenplays.',

  'title': 'dramatron',

  'link': 'https://github.com/deepmind/dramatron',

  'engines': ['github'],

  'category': 'it'},

 {'snippet': 'Code for loralib, an implementation of "LoRA: Low-Rank' 

```
```markdown

### LoRA

- 链接：[LoRA](https://github.com/microsoft/LoRA)

- 引擎：github

- 类别：it

- 摘要：《LoRA》论文的代码。

### human-eval

- 链接：[human-eval](https://github.com/openai/human-eval)

- 引擎：github

- 类别：it

- 摘要：《Evaluating Large Language Models Trained on Code》论文的代码。

### Chain-of-ThoughtsPapers

- 链接：[Chain-of-ThoughtsPapers](https://github.com/Timothyxxx/Chain-of-ThoughtsPapers)

- 引擎：github

- 类别：it

- 摘要：始于《Chain of Thought Prompting Elicits Reasoning in Large Language Models》的趋势。

### mistral

- 链接：[mistral](https://github.com/stanford-crfm/mistral)

- 引擎：github

- 类别：it

- 摘要：Mistral 是一个透明且易于访问的大规模语言模型训练框架，使用 Hugging Face 🤗 Transformers 构建。

### prize

- 链接：[prize](https://github.com/inverse-scaling/prize)

- 引擎：github

- 类别：it

- 摘要：寻找导致大型语言模型显示逆比例缩放的任务的奖励。

### Optimus

- 链接：[Optimus](https://github.com/ChunyuanLI/Optimus)

- 引擎：github

- 类别：it

- 摘要：第一个大规模预训练 VAE 语言模型 Optimus。

### llm-seminar

- 链接：[llm-seminar](https://github.com/craffel/llm-seminar)

- 引擎：github

- 类别：it

- 摘要：北卡罗来纳大学教堂山分校 2022 年秋季的大型语言模型研讨会。

### ThoughtSource

- 链接：[ThoughtSource](https://github.com/OpenBioLink/ThoughtSource)

- 引擎：github

- 类别：it

- 摘要：一个关于大型语言模型链式思维推理相关数据和工具的中心开放资源。由 Samwald 研究小组开发：https://samwald.info/

### Awesome-LLM-Robotics

- 链接：[Awesome-LLM-Robotics](https://github.com/GT-RIPL/Awesome-LLM-Robotics)

- 引擎：github

- 类别：it

- 摘要：使用大型语言/多模型模型进行机器人学/RL 的论文、代码和相关网站的综合列表。

### biomedical

- 链接：[biomedical](https://github.com/bigscience-workshop/biomedical)

- 引擎：github

- 类别：it

- 摘要：用于大规模语言建模的生物医学训练数据的工具。

### ChatGPT-at-Home

- 链接：[ChatGPT-at-Home](https://github.com/Sentdex/ChatGPT-at-Home)

- 引擎：github

- 类别：it

- 摘要：ChatGPT @ Home：由 ChatGPT 编写的大型语言模型 (LLM) 聊天机器人应用。

### dust

- 链接：[dust](https://github.com/dust-tt/dust)

- 引擎：github

- 类别：it

- 摘要：设计和部署大型语言模型应用的工具。

### polyglot

- 链接：[polyglot](https://github.com/EleutherAI/polyglot)

- 引擎：github

- 类别：it

- 摘要：Polyglot：多语言中表现均衡的大型语言模型。

### LaViLa

- 链接：[LaViLa](https://github.com/facebookresearch/LaViLa)

- 引擎：github

- 类别：it

- 摘要：《Learning Video Representations from Large Language Models》的代码发布。

### smoothquant

- 链接：[smoothquant](https://github.com/mit-han-lab/smoothquant)

- 引擎：github

- 类别：it

- 摘要：SmoothQuant：用于大型语言模型的准确高效的后训练量化。

### xl-sum

- 链接：[xl-sum](https://github.com/csebuetnlp/xl-sum)

- 引擎：github

- 类别：it

- 摘要：这个存储库包含了《XL-Sum: Large-Scale Multilingual Abstractive Summarization for 44 Languages》一文中发表的代码、数据和模型，该论文发表在计算语言学协会的 Findings of the Association for Computational Linguistics: ACL-IJCNLP 2021 会议上。

```