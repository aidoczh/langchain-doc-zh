# Jina Reranker

本笔记展示了如何使用 Jina Reranker 进行文档压缩和检索。

```python
%pip install -qU langchain langchain-openai langchain-community langchain-text-splitters langchainhub
%pip install --upgrade --quiet  faiss
# 或者（取决于 Python 版本）
%pip install --upgrade --quiet  faiss_cpu
```

```python
# 用于打印文档的辅助函数
def pretty_print_docs(docs):
    print(
        f"\n{'-' * 100}\n".join(
            [f"Document {i+1}:\n\n" + d.page_content for i, d in enumerate(docs)]
        )
    )
```

## 设置基础向量存储检索器

让我们首先初始化一个简单的向量存储检索器，并存储 2023 年国情咨文（分块）。我们可以设置检索器以检索高数量（20）的文档。

##### 设置 Jina 和 OpenAI API 密钥

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
os.environ["JINA_API_KEY"] = getpass.getpass()
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings import JinaEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
documents = TextLoader(
    "../../how_to/state_of_the_union.txt",
).load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
embedding = JinaEmbeddings(model_name="jina-embeddings-v2-base-en")
retriever = FAISS.from_documents(texts, embedding).as_retriever(search_kwargs={"k": 20})
query = "What did the president say about Ketanji Brown Jackson"
docs = retriever.get_relevant_documents(query)
pretty_print_docs(docs)
```

## 使用 JinaRerank 进行重新排名

现在让我们用 ContextualCompressionRetriever 将我们的基础检索器包装起来，使用 Jina Reranker 作为压缩器。

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain_community.document_compressors import JinaRerank
compressor = JinaRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.get_relevant_documents(
    "What did the president say about Ketanji Jackson Brown"
)
```

```python
pretty_print_docs(compressed_docs)
```

## 使用 Jina Reranker 进行问答重新排名

```python
from langchain import hub
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
retrieval_qa_chat_prompt = hub.pull("langchain-ai/retrieval-qa-chat")
retrieval_qa_chat_prompt.pretty_print()
```

```output
================================ System Message ================================
Answer any use questions based solely on the context below:
<context>
{context}
</context>
============================= Messages Placeholder =============================
{chat_history}
================================ Human Message =================================
{input}
```

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
combine_docs_chain = create_stuff_documents_chain(llm, retrieval_qa_chat_prompt)
chain = create_retrieval_chain(compression_retriever, combine_docs_chain)
```

```python
chain.invoke({"input": query})
```