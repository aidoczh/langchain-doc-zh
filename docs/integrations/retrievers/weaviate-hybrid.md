# Weaviate 混合搜索

[Weaviate](https://weaviate.io/developers/weaviate) 是一个开源的向量数据库。

[混合搜索](https://weaviate.io/blog/hybrid-search-explained) 是一种技术，它结合了多种搜索算法，以提高搜索结果的准确性和相关性。它利用了基于关键词的搜索算法和向量搜索技术的最佳特点。

`Weaviate 中的混合搜索` 使用稀疏向量和密集向量来表示搜索查询和文档的含义和上下文。

这篇笔记展示了如何将 `Weaviate 混合搜索` 作为 LangChain 检索器使用。

设置检索器：

```python
%pip install --upgrade --quiet  weaviate-client
```

```python
import os
import weaviate
WEAVIATE_URL = os.getenv("WEAVIATE_URL")
auth_client_secret = (weaviate.AuthApiKey(api_key=os.getenv("WEAVIATE_API_KEY")),)
client = weaviate.Client(
    url=WEAVIATE_URL,
    additional_headers={
        "X-Openai-Api-Key": os.getenv("OPENAI_API_KEY"),
    },
)
```

```python
from langchain_community.retrievers import (
    WeaviateHybridSearchRetriever,
)
from langchain_core.documents import Document
```

```output
```

```python
retriever = WeaviateHybridSearchRetriever(
    client=client,
    index_name="LangChain",
    text_key="text",
    attributes=[],
    create_schema_if_missing=True,
)
```

添加一些数据：

```python
docs = [
    Document(
        metadata={
            "title": "Embracing The Future: AI Unveiled",
            "author": "Dr. Rebecca Simmons",
        },
        page_content="A comprehensive analysis of the evolution of artificial intelligence, from its inception to its future prospects. Dr. Simmons covers ethical considerations, potentials, and threats posed by AI.",
    ),
    Document(
        metadata={
            "title": "Symbiosis: Harmonizing Humans and AI",
            "author": "Prof. Jonathan K. Sterling",
        },
        page_content="Prof. Sterling explores the potential for harmonious coexistence between humans and artificial intelligence. The book discusses how AI can be integrated into society in a beneficial and non-disruptive manner.",
    ),
    Document(
        metadata={"title": "AI: The Ethical Quandary", "author": "Dr. Rebecca Simmons"},
        page_content="In her second book, Dr. Simmons delves deeper into the ethical considerations surrounding AI development and deployment. It is an eye-opening examination of the dilemmas faced by developers, policymakers, and society at large.",
    ),
    Document(
        metadata={
            "title": "Conscious Constructs: The Search for AI Sentience",
            "author": "Dr. Samuel Cortez",
        },
        page_content="Dr. Cortez takes readers on a journey exploring the controversial topic of AI consciousness. The book provides compelling arguments for and against the possibility of true AI sentience.",
    ),
    Document(
        metadata={
            "title": "Invisible Routines: Hidden AI in Everyday Life",
            "author": "Prof. Jonathan K. Sterling",
        },
        page_content="In his follow-up to 'Symbiosis', Prof. Sterling takes a look at the subtle, unnoticed presence and influence of AI in our everyday lives. It reveals how AI has become woven into our routines, often without our explicit realization.",
    ),
]
```

```python
retriever.add_documents(docs)
```

```output
['3a27b0a5-8dbb-4fee-9eba-8b6bc2c252be',
 'eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907',
 '7ebbdae7-1061-445f-a046-1989f2343d8f',
 'c2ab315b-3cab-467f-b23a-b26ed186318d',
 'b83765f2-e5d2-471f-8c02-c3350ade4c4f']
```

进行混合搜索：

```python
retriever.invoke("the ethical implications of AI")
```

```output
[Document(page_content='In her second book, Dr. Simmons delves deeper into the ethical considerations surrounding AI development and deployment. It is an eye-opening examination of the dilemmas faced by developers, policymakers, and society at large.', metadata={}),
 Document(page_content='A comprehensive analysis of the evolution of artificial intelligence, from its inception to its future prospects. Dr. Simmons covers ethical considerations, potentials, and threats posed by AI.', metadata={}),
 Document(page_content="In his follow-up to 'Symbiosis', Prof. Sterling takes a look at the subtle, unnoticed presence and influence of AI in our everyday lives. It reveals how AI has become woven into our routines, often without our explicit realization.", metadata={}),
 Document(page_content='Prof. Sterling explores the potential for harmonious coexistence between humans and artificial intelligence. The book discusses how AI can be integrated into society in a beneficial and non-disruptive manner.', metadata={})]
```

使用带有 where 过滤器的混合搜索：

```python
retriever.invoke(
    "AI integration in society",
    where_filter={
        "path": ["author"],
        "operator": "Equal",
        "valueString": "Prof. Jonathan K. Sterling",
    },
)
```

在他的著作《共生》之后，Sterling教授探讨了人类与人工智能和谐共处的潜力。该书讨论了如何以有益且不具破坏性的方式将人工智能整合到社会中。[20]

在他的后续著作中，Sterling教授审视了人工智能在我们日常生活中微妙而不为人察觉的存在和影响。这揭示了人工智能如何已经悄然融入我们的日常生活，往往在我们并没有明确意识到的情况下。[21]

```python
retriever.invoke(
    "AI integration in society",
    score=True,
)
```

Sterling教授探讨了人类与人工智能和谐共处的潜力。该书讨论了如何以有益且不具破坏性的方式将人工智能整合到社会中。[20]metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907 contributed 0.00819672131147541 to the score\n(hybrid) Document eeb9fd9b-a3ac-4d60-a55b-a63a25d3b907 contributed 0.00819672131147541 to the score', 'score': '0.016393442'}}

在他的后续著作中，Sterling教授审视了人工智能在我们日常生活中微妙而不为人察觉的存在和影响。这揭示了人工智能如何已经悄然融入我们的日常生活，往往在我们并没有明确意识到的情况下。[21]metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document b83765f2-e5d2-471f-8c02-c3350ade4c4f contributed 0.0078125 to the score\n(hybrid) Document b83765f2-e5d2-471f-8c02-c3350ade4c4f contributed 0.008064516129032258 to the score', 'score': '0.015877016'}}

在她的第二本书中，Simmons博士深入探讨了围绕人工智能开发和部署的伦理考量。这是对开发者、政策制定者和整个社会所面临的困境的深入审视。metadata={'_additional': {'explainScore': '(bm25)\n(hybrid) Document 7ebbdae7-1061-445f-a046-1989f2343d8f contributed 0.008064516129032258 to the score\n(hybrid) Document 7ebbdae7-1061-445f-a046-1989f2343d8f contributed 0.0078125 to the score', 'score': '0.015877016'}}

Simmons博士对人工智能的演变进行了全面分析，从其起源到未来的前景。Simmons涵盖了人工智能带来的伦理考量、潜力和威胁。metadata={'_additional': {'explainScore': '(vector) [-0.0071824766 -0.0006682752 0.001723625 -0.01897258 -0.0045127636 0.0024410256 -0.020503938 0.013768672 0.009520169 -0.037972264]...  \n(hybrid) Document 3a27b0a5-8dbb-4fee-9eba-8b6bc2c252be contributed 0.007936507936507936 to the score', 'score': '0.007936508'}}

```