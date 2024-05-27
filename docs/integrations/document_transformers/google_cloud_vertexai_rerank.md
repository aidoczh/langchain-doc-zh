# 谷歌云 Vertex AI 重新排序器

[Vertex Search Ranking API](https://cloud.google.com/generative-ai-app-builder/docs/ranking)是[Vertex AI Agent Builder](https://cloud.google.com/generative-ai-app-builder/docs/builder-apis)中独立的 API 之一。它接收一个文档列表，并根据文档与查询的相关性重新排列这些文档。与仅查看文档和查询的语义相似性的嵌入相比，排名 API 可以为文档对给定查询的回答程度提供精确分数。在检索到初始候选文档后，排名 API 可用于提高搜索结果的质量。

排名 API 是无状态的，因此在调用 API 之前无需对文档进行索引。您只需要传入查询和文档即可。这使得该 API 非常适合从任何文档检索器中重新排序文档。

更多信息，请参阅[Rank and rerank documents](https://cloud.google.com/generative-ai-app-builder/docs/ranking)。

```python
%pip install --upgrade --quiet langchain langchain-community langchain-google-community langchain-google-community[vertexaisearch] langchain-google-vertexai langchain-chroma langchain-text-splitters
```

### 设置

```python
PROJECT_ID = ""
REGION = ""
RANKING_LOCATION_ID = "global"  # @param {type:"string}
# 初始化 Vertex AI 的 GCP 项目
from google.cloud import aiplatform
aiplatform.init(project=PROJECT_ID, location=REGION)
```

### 加载和准备数据

在这个示例中，我们将使用[谷歌维基页面](https://en.wikipedia.org/wiki/Google)来演示 Vertex Ranking API 的工作原理。

我们使用标准的流水线 `加载 -> 分割 -> 嵌入数据`。

嵌入是使用[Vertex Embeddings API](https://cloud.google.com/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported_models)模型 - `textembedding-gecko@003` 创建的。

```python
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_google_vertexai import VertexAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
vectordb = None
# 加载维基页面
loader = WebBaseLoader("https://en.wikipedia.org/wiki/Google")
data = loader.load()
# 将文档分成块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=5)
splits = text_splitter.split_documents(data)
print(f"您的 {len(data)} 个文档已被分成 {len(splits)} 个块")
if vectordb is not None:  # 如果已存在 vectordb，则删除
    vectordb.delete_collection()
embedding = VertexAIEmbeddings(model_name="textembedding-gecko@003")
vectordb = Chroma.from_documents(documents=splits, embedding=embedding)
```

```output
您的 1 个文档已被分成 266 个块
```

```python
import pandas as pd
from langchain.retrievers.contextual_compression import ContextualCompressionRetriever
from langchain_google_community.vertex_rank import VertexAIRank
# 使用 SDK 管理器实例化 VertexAIReranker
reranker = VertexAIRank(
    project_id=PROJECT_ID,
    location_id=RANKING_LOCATION_ID,
    ranking_config="default_ranking_config",
    title_field="source",
    top_n=5,
)
basic_retriever = vectordb.as_retriever(search_kwargs={"k": 5})  # 获取前 5 个文档
# 使用 VertexAIRanker 作为重新排序器创建 ContextualCompressionRetriever
retriever_with_reranker = ContextualCompressionRetriever(
    base_compressor=reranker, base_retriever=basic_retriever
)
```

### 测试 Vertex Ranking API

让我们使用相同的查询分别查询 `basic_retriever` 和 `retriever_with_reranker`，并比较检索到的文档。

排名 API 接收来自 `basic_retriever` 的输入，并将其传递给排名 API。

排名 API 用于提高排名质量并确定一个分数，指示每个记录与查询的相关性。

您可以看到未排序和已排序文档之间的差异。排名 API 将语义相关性最高的文档移至 LLM 上下文窗口的顶部，从而帮助其形成更好的带有推理的答案。

```python
import pandas as pd
# 使用 basic_retriever 和 retriever_with_reranker 获取相关文档
query = "谷歌的名称是如何起源的？"
retrieved_docs = basic_retriever.invoke(query)
reranked_docs = retriever_with_reranker.invoke(query)
# 创建未排序和已排序文档的结果列表
unranked_docs_content = [docs.page_content for docs in retrieved_docs]
ranked_docs_content = [docs.page_content for docs in reranked_docs]
# 使用填充列表创建比较 DataFrame
comparison_df = pd.DataFrame(
    {
        "未排序文档": unranked_docs_content,
        "已排序文档": ranked_docs_content,
    }
)
comparison_df
```

让我们来检查一些重新排名的文档。我们发现检索器仍然会返回相关的 Langchain 类型[文档](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)，但在元数据字段中，我们还会从排名 API 中收到 `relevance_score`。

```python
for i in range(2):
    print(f"Document {i}")
    print(reranked_docs[i])
    print("----------------------------------------------------------\n")
```

```html
<style>
  pre {
      white-space: pre-wrap;
  }
</style>
```

```output
Document 0
page_content='The name "Google" originated from a misspelling of "googol",[211][212] which refers to the number represented by a 1 followed by one-hundred zeros. Page and Brin write in their original paper on PageRank:[33] "We chose our system name, Google, because it is a common spelling of googol, or 10100[,] and fits well with our goal of building very large-scale search engines." Having found its way increasingly into everyday language, the verb "google" was added to the Merriam Webster Collegiate Dictionary and the Oxford English Dictionary in 2006, meaning "to use the Google search engine to obtain information on the Internet."[213][214] Google\'s mission statement, from the outset, was "to organize the world\'s information and make it universally accessible and useful",[215] and its unofficial' metadata={'id': '2', 'relevance_score': 0.9800000190734863, 'source': 'https://en.wikipedia.org/wiki/Google'}
----------------------------------------------------------
Document 1
page_content='Eventually, they changed the name to Google; the name of the search engine was a misspelling of the word googol,[21][36][37] a very large number written 10100 (1 followed by 100 zeros), picked to signify that the search engine was intended to provide large quantities of information.[38]' metadata={'id': '1', 'relevance_score': 0.75, 'source': 'https://en.wikipedia.org/wiki/Google'}
----------------------------------------------------------
```

### 将所有内容整合在一起

这展示了一个完整的 RAG 链的示例，其中包括一个简单的提示模板，演示了如何使用 Vertex Ranking API 进行重新排名。

```python
from langchain.chains import LLMChain
from langchain_core.documents import Document
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_google_vertexai import VertexAI
llm = VertexAI(model_name="gemini-1.0-pro-002")
# 用 SDK 管理器实例化 VertexAIReranker
reranker = VertexAIRank(
    project_id=PROJECT_ID,
    location_id=RANKING_LOCATION_ID,
    ranking_config="default_ranking_config",
    title_field="source",  # 从现有文档中的元数据字段键
    top_n=5,
)
# k 的值也可以设置得更高，以调整性能
# 例如：文档数量：basic_retriever(100) -> reranker(5)
basic_retriever = vectordb.as_retriever(search_kwargs={"k": 5})  # 获取前 5 个文档
# 使用 VertexAIRanker 作为 Reranker 创建 ContextualCompressionRetriever
retriever_with_reranker = ContextualCompressionRetriever(
    base_compressor=reranker, base_retriever=basic_retriever
)
template = """
<context>
{context}
</context>
Question:
{query}
不要提供上下文之外的信息或重复你的发现。
Answer:
"""
prompt = PromptTemplate.from_template(template)
reranker_setup_and_retrieval = RunnableParallel(
    {"context": retriever_with_reranker, "query": RunnablePassthrough()}
)
chain = reranker_setup_and_retrieval | prompt | llm
```

```html
<style>
  pre {
      white-space: pre-wrap;
  }
</style>
```

```python
query = "how did the name google originate?"
```

```html
<style>
  pre {
      white-space: pre-wrap;
  }
</style>
```

```python
chain.invoke(query)
```

```html
<style>
  pre {
      white-space: pre-wrap;
  }
</style>
```

```output
'The name "Google" originated as a misspelling of the word "googol," a mathematical term for the number 1 followed by 100 zeros. Larry Page and Sergey Brin, the founders of Google, chose the name because it reflected their goal of building a search engine that could handle massive amounts of information. \n'
```