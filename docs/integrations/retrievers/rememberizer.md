# 记忆增强器

>[记忆增强器](https://rememberizer.ai/)是由 SkyDeck AI 公司创建的一项用于 AI 应用的知识增强服务。

本笔记展示了如何从 `Rememberizer` 中检索文档，并将其转换为下游使用的文档格式。

# 准备工作

您需要一个 API 密钥：您可以在 [https://rememberizer.ai](https://rememberizer.ai/) 创建通用知识后获取一个 API 密钥。一旦您获得了 API 密钥，您必须将其设置为环境变量 `REMEMBERIZER_API_KEY`，或者在初始化 `RememberizerRetriever` 时将其作为 `rememberizer_api_key` 传递。

`RememberizerRetriever` 有以下参数：

- 可选参数 `top_k_results`：默认值为 10。使用它来限制返回的文档数量。

- 可选参数 `rememberizer_api_key`：如果您没有设置环境变量 `REMEMBERIZER_API_KEY`，则此参数是必需的。

`get_relevant_documents()` 有一个参数 `query`：用于在 `Rememberizer.ai` 的通用知识中查找文档的自由文本。

# 示例

## 基本用法

```python
# 设置 API 密钥
from getpass import getpass
REMEMBERIZER_API_KEY = getpass()
```

```python
import os
from langchain_community.retrievers import RememberizerRetriever
os.environ["REMEMBERIZER_API_KEY"] = REMEMBERIZER_API_KEY
retriever = RememberizerRetriever(top_k_results=5)
```

```python
docs = retriever.get_relevant_documents(query="How does Large Language Models works?")
```

```python
docs[0].metadata  # 文档的元信息
```

```output
{'id': 13646493,
 'document_id': '17s3LlMbpkTk0ikvGwV0iLMCj-MNubIaP',
 'name': 'What is a large language model (LLM)_ _ Cloudflare.pdf',
 'type': 'application/pdf',
 'path': '/langchain/What is a large language model (LLM)_ _ Cloudflare.pdf',
 'url': 'https://drive.google.com/file/d/17s3LlMbpkTk0ikvGwV0iLMCj-MNubIaP/view',
 'size': 337089,
 'created_time': '',
 'modified_time': '',
 'indexed_on': '2024-04-04T03:36:28.886170Z',
 'integration': {'id': 347, 'integration_type': 'google_drive'}}
```

```python
print(docs[0].page_content[:400])  # 文档内容
```

```output
before, or contextualized in new ways. on some level they " understand " semantics in that they can associate words and concepts by their meaning, having seen them grouped together in that way millions or billions of times. how developers can quickly start building their own llms to build llm applications, developers need easy access to multiple data sets, and they need places for those data sets
```

## 链式使用

```python
OPENAI_API_KEY = getpass()
```

```python
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model_name="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```

```python
questions = [
    "What is RAG?",
    "How does Large Language Models works?",
]
chat_history = []
for question in questions:
    result = qa.invoke({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **Question**: {question} \n")
    print(f"**Answer**: {result['answer']} \n")
```

```output
-> **Question**: What is RAG? 
**Answer**: RAG 是检索增强生成（Retrieval-Augmented Generation）的缩写。这是一个 AI 框架，它从外部知识库中检索事实，以增强大型语言模型（LLMs）生成的响应，提供最新和准确的信息。该框架帮助用户理解 LLMs 的生成过程，并确保模型可以访问可靠的信息来源。
-> **Question**: How does Large Language Models works? 
**Answer**: 大型语言模型（LLMs）通过分析大量的语言数据集来理解和生成人类语言文本。它们建立在机器学习的基础上，具体来说是深度学习，涉及训练程序以识别数据特征而无需人为干预。LLMs 使用神经网络，特别是变压器模型，来理解人类语言中的上下文，使它们更擅长解释语言，即使在模糊或新的语境中也是如此。开发人员可以通过访问多个数据集并使用 Cloudflare 的 Vectorize 和 Cloudflare Workers AI 平台等服务，快速开始构建自己的 LLMs。
```