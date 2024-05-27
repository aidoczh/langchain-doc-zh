# Cohere RAG

[Cohere](https://cohere.ai/about) 是一家加拿大初创公司，提供自然语言处理模型，帮助企业改善人机交互。

这篇笔记介绍了如何开始使用 `Cohere RAG` 检索器。这使您能够利用搜索各种连接器上的文档，或者通过提供自己的文档。

```python
import getpass
import os
os.environ["COHERE_API_KEY"] = getpass.getpass()
```

```python
from langchain_cohere import ChatCohere
from langchain_community.retrievers import CohereRagRetriever
from langchain_core.documents import Document
```

```python
rag = CohereRagRetriever(llm=ChatCohere())
```

```python
def _pretty_print(docs):
    for doc in docs:
        print(doc.metadata)
        print("\n\n" + doc.page_content)
        print("\n\n" + "-" * 30 + "\n\n")
```

```python
_pretty_print(rag.invoke("What is cohere ai?"))
```

```output
{'id': 'web-search_4:0', 'snippet': 'AI startup Cohere, now valued at over $2.1B, raises $270M\n\nKyle Wiggers 4 months\n\nIn a sign that there’s plenty of cash to go around for generative AI startups, Cohere, which is developing an AI model ecosystem for the enterprise, today announced that it raised $270 million as part of its Series C round.\n\nReuters reported earlier in the year that Cohere was in talks to raise “hundreds of millions” of dollars at a valuation of upward of just over $6 billion. If there’s credence to that reporting, Cohere appears to have missed the valuation mark substantially; a source familiar with the matter tells TechCrunch that this tranche values the company at between $2.1 billion and $2.2 billion.', 'title': 'AI startup Cohere, now valued at over $2.1B, raises $270M | TechCrunch', 'url': 'https://techcrunch.com/2023/06/08/ai-startup-cohere-now-valued-at-over-2-1b-raises-270m/'}
AI 初创公司 Cohere，现在估值超过 21 亿美元，筹集了 2.7 亿美元
Kyle Wiggers 4 个月
迹象表明，生成式人工智能初创公司有大量资金可供使用，Cohere 正在为企业开发人工智能模型生态系统，今天宣布其在 C 轮融资中筹集了 2.7 亿美元。
路透社今年早些时候报道说，Cohere 正在谈判筹集“数亿美元”，估值超过 60 亿美元。如果这一报道是可信的，Cohere 显然大幅错过了估值标记；知情人士告诉 TechCrunch，这一轮融资将公司估值定为 21 亿美元至 22 亿美元。
------------------------------
{'id': 'web-search_9:0', 'snippet': 'Cohere is a Canadian multinational technology company focused on artificial intelligence for the enterprise, specializing in large language models. Cohere was founded in 2019 by Aidan Gomez, Ivan Zhang, and Nick Frosst, and is headquartered in Toronto and San Francisco, with offices in Palo Alto and London.\n\nIn 2017, a team of researchers at Google Brain, which included Aidan Gomez, published a paper called "Attention is All You Need," which introduced the transformer machine learning architecture, setting state-of-the-art performance on a variety of natural language processing tasks. In 2019, Gomez and Nick Frosst, another researcher at Google Brain, founded Cohere along with Ivan Zhang, with whom Gomez had done research at FOR.ai. All of the co-founders attended University of Toronto.', 'title': 'Cohere - Wikipedia', 'url': 'https://en.wikipedia.org/wiki/Cohere'}
Cohere 是一家专注于为企业提供人工智能技术的加拿大跨国科技公司，专门研发大型语言模型。Cohere 成立于 2019 年，由 Aidan Gomez、Ivan Zhang 和 Nick Frosst 创立，总部位于多伦多和旧金山，在帕洛阿尔托和伦敦设有办事处。
2017 年，Google Brain 的研究团队（包括 Aidan Gomez）发表了一篇名为“Attention is All You Need”的论文，介绍了变压器机器学习架构，在各种自然语言处理任务中取得了最先进的性能。2019 年，Gomez 和另一位 Google Brain 的研究人员 Nick Frosst 与 Ivan Zhang 共同创立了 Cohere，Gomez 与 Zhang 曾在 FOR.ai 进行过研究。所有联合创始人都毕业于多伦多大学。
------------------------------
{'id': 'web-search_8:2', 'snippet': 'Cofounded by Aidan Gomez, a Google Brain alum and coauthor of the seminal transformer research paper, Cohere describes itself as being “on a mission to transform enterprises and their products with AI to unlock a more intuitive way to generate, search, and summarize information than ever before.” One key element of Cohere’s approach is its focus on data protection, deploying its models inside enterprises’ secure data environment.\n\n“We are both independent and cloud-agnostic, meaning we are not beholden to any one tech company and empower enterprises to implement customized AI solutions on the cloud of their choosing, or even on-premises,” says Martin Kon, COO and president of Cohere.', 'title': 'McKinsey and Cohere collaborate to transform clients with enterprise generative AI', 'url': 'https://www.mckinsey.com/about-us/new-at-mckinsey-blog/mckinsey-and-cohere-collaborate-to-transform-clients-with-enterprise-generative-ai'}
由 Google Brain 校友、开创性变压器研究论文的合著者 Aidan Gomez 共同创立，Cohere 自称“致力于通过人工智能改变企业及其产品，解锁比以往更直观的生成、搜索和总结信息方式”。Cohere 方法的一个关键要素是其专注于数据保护，将其模型部署在企业安全数据环境中。
Cohere 的 COO 兼总裁 Martin Kon 表示：“我们既独立又云无关，这意味着我们不受任何科技公司的约束，赋予企业在其选择的云上或甚至本地部署定制的人工智能解决方案的能力。”
------------------------------
```

```python
_pretty_print(await rag.ainvoke("Cohere 是什么？"))  # 异步版本
```

```output
{'id': 'web-search_9:0', 'snippet': 'Cohere 是一家加拿大跨国科技公司，专注于为企业提供人工智能服务，专门研发大型语言模型。Cohere 成立于2019年，创始人包括 Aidan Gomez、Ivan Zhang 和 Nick Frosst，总部位于多伦多和旧金山，并在帕洛阿尔托和伦敦设有办事处。\n\n2017年，Google Brain 的研究团队（包括 Aidan Gomez）发表了一篇名为“Attention is All You Need”的论文，介绍了变压器机器学习架构，在各种自然语言处理任务中取得了最先进的性能。2019年，Gomez 和另一位 Google Brain 的研究人员 Nick Frosst 与 Ivan Zhang 共同创立了 Cohere，Gomez 与 Zhang 曾在 FOR.ai 进行过研究。所有的联合创始人都毕业于多伦多大学。', 'title': 'Cohere - Wikipedia', 'url': 'https://en.wikipedia.org/wiki/Cohere'}
Cohere 是一家加拿大跨国科技公司，专注于为企业提供人工智能服务，专门研发大型语言模型。Cohere 成立于2019年，创始人包括 Aidan Gomez、Ivan Zhang 和 Nick Frosst，总部位于多伦多和旧金山，并在帕洛阿尔托和伦敦设有办事处。
2017年，Google Brain 的研究团队（包括 Aidan Gomez）发表了一篇名为“Attention is All You Need”的论文，介绍了变压器机器学习架构，在各种自然语言处理任务中取得了最先进的性能。2019年，Gomez 和另一位 Google Brain 的研究人员 Nick Frosst 与 Ivan Zhang 共同创立了 Cohere，Gomez 与 Zhang 曾在 FOR.ai 进行过研究。所有的联合创始人都毕业于多伦多大学。
------------------------------
{'id': 'web-search_8:2', 'snippet': '由 Google Brain 校友 Aidan Gomez 共同创立，他是开创性变压器研究论文的合著者之一。Cohere 自称“致力于通过人工智能改变企业和产品，以解锁比以往更直观的方式生成、搜索和总结信息。” Cohere 方法的一个关键元素是其专注于数据保护，将其模型部署在企业安全数据环境中。\n\nCohere 的 COO 兼总裁 Martin Kon 表示：“我们既独立又云不可知，这意味着我们不受任何科技公司的约束，赋予企业在其选择的云上或甚至本地实施定制的人工智能解决方案的能力。”', 'title': 'McKinsey and Cohere collaborate to transform clients with enterprise generative AI', 'url': 'https://www.mckinsey.com/about-us/new-at-mckinsey-blog/mckinsey-and-cohere-collaborate-to-transform-clients-with-enterprise-generative-ai'}
由 Google Brain 校友 Aidan Gomez 共同创立，他是开创性变压器研究论文的合著者之一。Cohere 自称“致力于通过人工智能改变企业和产品，以解锁比以往更直观的方式生成、搜索和总结信息。” Cohere 方法的一个关键元素是其专注于数据保护，将其模型部署在企业安全数据环境中。
Cohere 的 COO 兼总裁 Martin Kon 表示：“我们既独立又云不可知，这意味着我们不受任何科技公司的约束，赋予企业在其选择的云上或甚至本地实施定制的人工智能解决方案的能力。”
------------------------------
{'id': 'web-search_4:0', 'snippet': '人工智能初创公司 Cohere，目前估值超过 21 亿美元，筹集了 2.7 亿美元\n\nKyle Wiggers 4 个月前\n\nCohere 是一家为企业开发人工智能模型生态系统的公司，今天宣布作为 C 轮融资的一部分筹集了 2.7 亿美元。这表明对于生成式人工智能初创公司来说，有大量资金可用。今年早些时候，路透社报道称 Cohere 正在就估值超过 60 亿美元的“数亿美元”融资进行谈判。如果这一报道属实，Cohere 似乎大幅错过了估值标记；知情人士告诉 TechCrunch，这一轮融资将公司估值定在 21 亿至 22 亿美元之间。', 'title': 'AI startup Cohere, now valued at over $2.1B, raises $270M | TechCrunch', 'url': 'https://techcrunch.com/2023/06/08/ai-startup-cohere-now-valued-at-over-2-1b-raises-270m/'}
人工智能初创公司 Cohere，目前估值超过 21 亿美元，筹集了 2.7 亿美元
Kyle Wiggers 4 个月前
Cohere 是一家为企业开发人工智能模型生态系统的公司，今天宣布作为 C 轮融资的一部分筹集了 2.7 亿美元。
据路透社早些时候报道，Cohere正在就筹集“数亿美元”资金进行谈判，估值超过60亿美元。如果这一报道属实，Cohere似乎大幅偏离了估值标记；知情人士告诉TechCrunch，这一轮融资将使该公司估值在21亿至22亿美元之间。
```

```python
docs = rag.invoke(
    "Does langchain support cohere RAG?",
    source_documents=[
        Document(page_content="Langchain supports cohere RAG!"),
        Document(page_content="The sky is blue!"),
    ],
)
_pretty_print(docs)
```

```output
{'id': 'doc-0', 'snippet': 'Langchain supports cohere RAG!'}
Langchain supports cohere RAG!
```