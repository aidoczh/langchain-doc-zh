# Activeloop Deep Memory
[Activeloop Deep Memory](https://docs.activeloop.ai/performance-features/deep-memory) 是一套工具，可以帮助您优化您的 Vector Store，以适应您的用例，并在 LLM 应用中实现更高的准确性。
最近，`检索增强生成` (`RAG`) 技术引起了广泛关注。随着先进的 RAG 技术和代理的出现，RAG 的潜力也在不断扩大。然而，将 RAG 集成到生产环境中可能面临一些挑战。在实施 RAG 时需要考虑的主要因素是准确性（召回率）、成本和延迟。对于基本的用例，OpenAI 的 Ada 模型配合简单的相似性搜索可以产生令人满意的结果。然而，对于在搜索过程中需要更高准确性或召回率的情况，可能需要采用更高级的检索技术。这些方法可能涉及不同的数据块大小、多次重写查询等，可能会增加延迟和成本。Activeloop 的 [Deep Memory](https://www.activeloop.ai/resources/use-deep-memory-to-boost-rag-apps-accuracy-by-up-to-22/) 是 `Activeloop Deep Lake` 用户可以使用的一个功能，它通过引入一个小型神经网络层来训练，将用户查询与语料库中相关数据进行匹配。虽然这种添加在搜索过程中会带来最小的延迟，但它可以将检索准确性提高多达 27%，并且成本效益高且易于使用，无需额外的高级 RAG 技术。
在本教程中，我们将解析 `DeepLake` 文档，并创建一个能够回答文档中问题的 RAG 系统。
## 1. 数据集创建
我们将使用 `BeautifulSoup` 库和 LangChain 的文档解析器（如 `Html2TextTransformer`、`AsyncHtmlLoader`）来解析 activeloop 的文档。因此，我们需要安装以下库：
```python
%pip install --upgrade --quiet  tiktoken langchain-openai python-dotenv datasets langchain deeplake beautifulsoup4 html2text ragas
```
您还需要创建一个 [Activeloop](https://activeloop.ai) 账户。
```python
ORG_ID = "..."
```
```python
from langchain.chains import RetrievalQA
from langchain_community.vectorstores import DeepLake
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
```
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("请输入您的 OpenAI API 密钥：")
os.environ["ACTIVELOOP_TOKEN"] = getpass.getpass("请输入您的 ActiveLoop API 密钥：")
token = os.getenv("ACTIVELOOP_TOKEN")
openai_embeddings = OpenAIEmbeddings()
```
```python
db = DeepLake(
    dataset_path=f"hub://{ORG_ID}/deeplake-docs-deepmemory",
    embedding=openai_embeddings,
    runtime={"tensor_db": True},
    token=token,
    read_only=False,
)
```
使用 `BeautifulSoup` 解析网页中的所有链接：
```python
from urllib.parse import urljoin
import requests
from bs4 import BeautifulSoup
def get_all_links(url):
    response = requests.get(url)
    if response.status_code != 200:
        print(f"无法获取页面：{url}")
        return []
    soup = BeautifulSoup(response.content, "html.parser")
    links = [
        urljoin(url, a["href"]) for a in soup.find_all("a", href=True) if a["href"]
    ]
    return links
base_url = "https://docs.deeplake.ai/en/latest/"
all_links = get_all_links(base_url)
```
加载数据：
```python
from langchain_community.document_loaders.async_html import AsyncHtmlLoader
loader = AsyncHtmlLoader(all_links)
docs = loader.load()
```
将数据转换为可读格式：
```python
from langchain_community.document_transformers import Html2TextTransformer
html2text = Html2TextTransformer()
docs_transformed = html2text.transform_documents(docs)
```
现在，让我们进一步将文档分块，因为其中一些文档包含太多文本：
```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
chunk_size = 4096
docs_new = []
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size,
)
for doc in docs_transformed:
    if len(doc.page_content) < chunk_size:
        docs_new.append(doc)
    else:
        docs = text_splitter.create_documents([doc.page_content])
        docs_new.extend(docs)
```
填充 VectorStore：
```python
docs = db.add_documents(docs_new)
```
## 2. 生成合成查询并训练 Deep Memory
接下来的步骤是训练一个 deep_memory 模型，它将会将用户的查询与您已经拥有的数据集进行对齐。如果您还没有任何用户查询，不用担心，我们将使用 LLM 来生成它们！
#### TODO: 添加图片
上面我们展示了 deep_memory 的整体架构。因此，您可以看到，为了训练它，您需要相关性、查询以及语料库数据（我们想要查询的数据）。语料库数据已经在上一节中填充，这里我们将生成问题和相关性。
1. `questions` - 是一组文本字符串，其中每个字符串代表一个查询
2. `relevance` - 包含每个问题的真实链接。可能有几个文档包含了给定问题的答案。因此，相关性是 `List[List[tuple[str, float]]]`，外部列表代表查询，内部列表代表相关文档。元组包含字符串和浮点数对，其中字符串代表源文档的 id（对应数据集中的 `id` 张量），而浮点数对应当前文档与问题的相关程度。
现在，让我们生成合成问题和相关性：
```python
from typing import List
from langchain.chains.openai_functions import create_structured_output_chain
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.prompts import ChatPromptTemplate, HumanMessagePromptTemplate
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field
# 如果存在数据集文档和 id（可选，您也可以进行摄取）
docs = db.vectorstore.dataset.text.data(fetch_chunks=True, aslist=True)["value"]
ids = db.vectorstore.dataset.id.data(fetch_chunks=True, aslist=True)["value"]
# 如果我们显式传入一个模型，我们需要确保它支持 OpenAI 的函数调用 API。
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
class Questions(BaseModel):
    """关于一个人的身份信息。"""
    question: str = Field(..., description="关于文本的问题")
prompt_msgs = [
    SystemMessage(
        content="您是一个世界级专家，可以根据提供的上下文生成问题。您要确保问题可以由文本回答。"
    ),
    HumanMessagePromptTemplate.from_template(
        "使用给定的文本从以下输入中生成问题：{input}"
    ),
    HumanMessage(content="提示：确保以正确的格式回答"),
]
prompt = ChatPromptTemplate(messages=prompt_msgs)
chain = create_structured_output_chain(Questions, llm, prompt, verbose=True)
text = "# 理解幻觉和偏见 ## **介绍** 在本课中，我们将介绍 LLM 中的 **幻觉** 概念，突出它们对 AI 应用的影响，并演示如何使用检索器的架构来减轻它们。我们还将通过示例探讨 LLM 中的 **偏见**。"
questions = chain.run(input=text)
print(questions)
```
```python
import random
from langchain_openai import OpenAIEmbeddings
from tqdm import tqdm
def generate_queries(docs: List[str], ids: List[str], n: int = 100):
    questions = []
    relevances = []
    pbar = tqdm(total=n)
    while len(questions) < n:
        # 1. 随机选择一段文本和相关性 id
        r = random.randint(0, len(docs) - 1)
        text, label = docs[r], ids[r]
        # 2. 生成查询并分配相关性 id
        generated_qs = [chain.run(input=text).question]
        questions.extend(generated_qs)
        relevances.extend([[(label, 1)] for _ in generated_qs])
        pbar.update(len(generated_qs))
        if len(questions) % 10 == 0:
            print(f"q: {len(questions)}")
    return questions[:n], relevances[:n]
chain = create_structured_output_chain(Questions, llm, prompt, verbose=False)
questions, relevances = generate_queries(docs, ids, n=200)
train_questions, train_relevances = questions[:100], relevances[:100]
test_questions, test_relevances = questions[100:], relevances[100:]
```
现在我们创建了 100 个训练查询以及 100 个用于测试的查询。现在让我们训练 deep_memory：
```python
job_id = db.vectorstore.deep_memory.train(
    queries=train_questions,
    relevance=train_relevances,
)
```
让我们跟踪训练进度：
```python
db.vectorstore.deep_memory.status("6538939ca0b69a9ca45c528c")
```
```output
--------------------------------------------------------------
|                  6538e02ecda4691033a51c5b                  |
--------------------------------------------------------------
| status                     | completed                     |
--------------------------------------------------------------
| progress                   | eta: 1.4 seconds              ||                            | recall@10: 79.00% (+34.00%)   |
--------------------------------------------------------------
| results                    | recall@10: 79.00% (+34.00%)   |
--------------------------------------------------------------
```
## 3. 评估深度记忆性能
太棒了，我们已经训练好了模型！它在召回率上显示了一些实质性的改进，但现在我们该如何使用它并在未见过的新数据上进行评估呢？在本节中，我们将深入探讨模型评估和推理部分，并看看如何在 LangChain 中使用它以提高检索准确性。
### 3.1 深度记忆评估
首先，我们可以使用 deep_memory 内置的评估方法。
它计算了几个 `召回率` 指标。
可以很容易地用几行代码来完成。
```python
recall = db.vectorstore.deep_memory.evaluate(
    queries=test_questions,
    relevance=test_relevances,
)
```
```output
嵌入查询花费了 0.81 秒
---- 在没有模型的情况下进行评估 ----
召回率@1: 9.0%
召回率@3: 19.0%
召回率@5: 24.0%
召回率@10: 42.0%
召回率@50: 93.0%
召回率@100: 98.0%
---- 在有模型的情况下进行评估 ----
召回率@1: 19.0%
召回率@3: 42.0%
召回率@5: 49.0%
召回率@10: 69.0%
召回率@50: 97.0%
召回率@100: 97.0%
```
它在未见的测试数据集上也显示出了相当大的改进！！！
### 3.2 深度记忆 + RAGas
```python
from ragas.langchain import RagasEvaluatorChain
from ragas.metrics import (
    context_recall,
)
```
让我们将召回率转换为基本事实：
```python
def convert_relevance_to_ground_truth(docs, relevance):
    ground_truths = []
    for rel in relevance:
        ground_truth = []
        for doc_id, _ in rel:
            ground_truth.append(docs[doc_id])
        ground_truths.append(ground_truth)
    return ground_truths
```
```python
ground_truths = convert_relevance_to_ground_truth(docs, test_relevances)
for deep_memory in [False, True]:
    print("\n使用 deep_memory 进行评估 =", deep_memory)
    print("===================================")
    retriever = db.as_retriever()
    retriever.search_kwargs["deep_memory"] = deep_memory
    qa_chain = RetrievalQA.from_chain_type(
        llm=ChatOpenAI(model="gpt-3.5-turbo"),
        chain_type="stuff",
        retriever=retriever,
        return_source_documents=True,
    )
    metrics = {
        "context_recall_score": 0,
    }
    eval_chains = {m.name: RagasEvaluatorChain(metric=m) for m in [context_recall]}
    for question, ground_truth in zip(test_questions, ground_truths):
        result = qa_chain({"query": question})
        result["ground_truths"] = ground_truth
        for name, eval_chain in eval_chains.items():
            score_name = f"{name}_score"
            metrics[score_name] += eval_chain(result)[score_name]
    for metric in metrics:
        metrics[metric] /= len(test_questions)
        print(f"{metric}: {metrics[metric]}")
    print("===================================")
```
```output
使用 deep_memory 进行评估 = False
===================================
context_recall_score: 0.3763423145
===================================
使用 deep_memory 进行评估 = True
===================================
context_recall_score: 0.5634545323
===================================
```
### 3.3 深度记忆推理
#### TODO: 添加图片
使用 deep_memory
```python
retriever = db.as_retriever()
retriever.search_kwargs["deep_memory"] = True
retriever.search_kwargs["k"] = 10
query = "Deamination of cytidine to uridine on the minus strand of viral DNA results in catastrophic G-to-A mutations in the viral genome."
qa = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-4"), chain_type="stuff", retriever=retriever
)
print(qa.run(query))
```
```output
'video_seq' 张量的基本 htype 是 'video'。
```
不使用 deep_memory
```python
retriever = db.as_retriever()
retriever.search_kwargs["deep_memory"] = False
retriever.search_kwargs["k"] = 10
query = "Deamination of cytidine to uridine on the minus strand of viral DNA results in catastrophic G-to-A mutations in the viral genome."
qa = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-4"), chain_type="stuff", retriever=retriever
)
qa.run(query)
```
```output
文本未提供关于 'video_seq' 张量的基本 htype 的信息。
```
### 3.4 深度记忆成本节约
深度记忆提高了检索准确性，而不会改变您现有的工作流程。此外，通过减少输入到 LLM 中的 top_k 值，您可以显著降低推理成本，减少令牌使用量。