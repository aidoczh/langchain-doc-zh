# Voyage AI

[Voyage AI](https://www.voyageai.com/) 提供尖端的嵌入/向量化模型。

让我们加载 Voyage AI Embedding 类。(使用 `pip install langchain-voyageai` 安装 LangChain 合作伙伴包)

```python
from langchain_voyageai import VoyageAIEmbeddings
```

Voyage AI 利用 API 密钥来监控使用情况和管理权限。要获取您的密钥，请在我们的[主页](https://www.voyageai.com)上创建一个帐户。然后，使用您的 API 密钥创建一个 VoyageEmbeddings 模型。您可以使用以下任何模型：([来源](https://docs.voyageai.com/docs/embeddings)):

- `voyage-large-2` (默认)

- `voyage-code-2`

- `voyage-2`

- `voyage-law-2`

- `voyage-large-2-instruct`

```python
embeddings = VoyageAIEmbeddings(
    voyage_api_key="[您的 Voyage API 密钥]", model="voyage-law-2"
)
```

准备文档并使用 `embed_documents` 获取它们的嵌入。

```python
documents = [
    "Caching embeddings enables the storage or temporary caching of embeddings, eliminating the necessity to recompute them each time.",
    "An LLMChain is a chain that composes basic LLM functionality. It consists of a PromptTemplate and a language model (either an LLM or chat model). It formats the prompt template using the input key values provided (and also memory key values, if available), passes the formatted string to LLM and returns the LLM output.",
    "A Runnable represents a generic unit of work that can be invoked, batched, streamed, and/or transformed.",
```

```python
documents_embds = embeddings.embed_documents(documents)
```

```python
documents_embds[0][:5]
```

```output
[0.0562174916267395,
 0.018221192061901093,
 0.0025736060924828053,
 -0.009720131754875183,
 0.04108370840549469]
```

同样，使用 `embed_query` 嵌入查询。

```python
query = "What's an LLMChain?"
```

```python
query_embd = embeddings.embed_query(query)
```

```python
query_embd[:5]
```

```output
[-0.0052348352037370205,
 -0.040072452276945114,
 0.0033957737032324076,
 0.01763271726667881,
 -0.019235141575336456]
```

## 一种极简的检索系统

嵌入的主要特点是两个嵌入之间的余弦相似度捕获了相应原始段落的语义相关性。这使我们能够使用嵌入进行语义检索/搜索。

我们可以基于余弦相似度在文档嵌入中找到几个最接近的嵌入，并使用 LangChain 的 `KNNRetriever` 类检索相应的文档。

```python
from langchain_community.retrievers import KNNRetriever
retriever = KNNRetriever.from_texts(documents, embeddings)
# 检索最相关的文档
result = retriever.invoke(query)
top1_retrieved_doc = result[0].page_content  # 返回检索到的最相关结果
print(top1_retrieved_doc)
```

```output
An LLMChain is a chain that composes basic LLM functionality. It consists of a PromptTemplate and a language model (either an LLM or chat model). It formats the prompt template using the input key values provided (and also memory key values, if available), passes the formatted string to LLM and returns the LLM output.
```