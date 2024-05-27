

# Cohere重新排序器

[Cohere](https://cohere.ai/about) 是一家加拿大初创公司，提供自然语言处理模型，帮助公司改善人机交互。

这篇笔记展示了如何在检索器中使用[Cohere的重新排序端点](https://docs.cohere.com/docs/reranking)。这是在[ContextualCompressionRetriever](/docs/how_to/contextual_compression)的思想基础上构建的。

```python
%pip install --upgrade --quiet  cohere
```

```python
%pip install --upgrade --quiet  faiss
# 或者（取决于Python版本）
%pip install --upgrade --quiet  faiss-cpu
```

```python
# 获取新的令牌：https://dashboard.cohere.ai/
import getpass
import os
os.environ["COHERE_API_KEY"] = getpass.getpass("Cohere API Key:")
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

## 设置基本向量存储检索器

让我们从初始化一个简单的向量存储检索器开始，并存储2023年国情咨文（分块）。我们可以设置检索器以检索高数量（20）的文档。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings import CohereEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
retriever = FAISS.from_documents(texts, CohereEmbeddings()).as_retriever(
    search_kwargs={"k": 20}
)
query = "总统在演讲中提到了Ketanji Brown Jackson什么"
docs = retriever.invoke(query)
pretty_print_docs(docs)
```

```output
文档 1：
总统最严肃的宪法责任之一是提名人选担任美国最高法院的法官。
4天前，我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智囊，将延续布雷耶法官的卓越传统。
----------------------------------------------------------------------------------------------------
文档 2：
我们不能让这种情况发生。
今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。顺便再通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个致力于为国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
----------------------------------------------------------------------------------------------------
文档 3：
正如我去年所说，特别是对我们年轻的跨性别美国人，作为总统，我将永远支持你们，让你们做自己，实现上帝赋予你们的潜力。
尽管我们似乎从不达成一致，但事实并非如此。去年，我签署了80项跨党派法案。从防止政府关门到保护亚裔免受仍然普遍存在的仇恨犯罪，再到改革军事司法。
----------------------------------------------------------------------------------------------------
（后续文档内容省略）
```

## 使用 CohereRerank 进行重新排序

现在让我们用 `ContextualCompressionRetriever` 包装我们的基础检索器。我们将添加一个 `CohereRerank`，使用 Cohere 重新排序端点来重新排序返回的结果。

```python
from langchain.retrievers.contextual_compression import ContextualCompressionRetriever
from langchain_cohere import CohereRerank
from langchain_community.llms import Cohere
llm = Cohere(temperature=0)
compressor = CohereRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "What did the president say about Ketanji Jackson Brown"
)
pretty_print_docs(compressed_docs)
```

当然，你可以在问答流水线中使用这个检索器。

```python
from langchain.chains import RetrievalQA
```

```python
chain = RetrievalQA.from_chain_type(
    llm=Cohere(temperature=0), retriever=compression_retriever
)
```

```python
chain({"query": query})
```

```output
{'query': 'What did the president say about Ketanji Brown Jackson',
 'result': " The president speaks highly of Ketanji Brown Jackson, stating that she is one of the nation's top legal minds, and will continue the legacy of excellence of Justice Breyer. The president also mentions that he worked with her family and that she comes from a family of public school educators and police officers. Since her nomination, she has received support from various groups, including the Fraternal Order of Police and judges from both major political parties. \n\nWould you like me to extract another sentence from the provided text? "}
```