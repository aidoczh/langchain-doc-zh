# 如何使用 MultiVector 检索器

通常情况下，为每个文档存储多个向量是有益的。有多种情况可以从中受益。LangChain 提供了一个基础的 `MultiVectorRetriever`，使得查询这种设置变得容易。其中很多复杂性在于如何为每个文档创建多个向量。本文档涵盖了创建这些向量和使用 `MultiVectorRetriever` 的一些常见方法。

创建每个文档多个向量的方法包括：

- 较小的块：将文档分割成较小的块，并嵌入这些块（这是 ParentDocumentRetriever）。

- 摘要：为每个文档创建一个摘要，将其与文档一起嵌入（或者替代文档）。

- 假设问题：创建每个文档适合回答的假设问题，将其与文档一起嵌入（或者替代文档）。

请注意，这也可以通过另一种方法来添加嵌入 - 手动添加。这很棒，因为您可以明确添加应导致检索到文档的问题或查询，从而使您更具控制。

```python
from langchain.retrievers.multi_vector import MultiVectorRetriever
```

```python
from langchain.storage import InMemoryByteStore
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
loaders = [
    TextLoader("../../paul_graham_essay.txt"),
    TextLoader("state_of_the_union.txt"),
]
docs = []
for loader in loaders:
    docs.extend(loader.load())
text_splitter = RecursiveCharacterTextSplitter(chunk_size=10000)
docs = text_splitter.split_documents(docs)
```

## 较小的块

通常情况下，检索较大的信息块，但嵌入较小的块可能很有用。这允许嵌入尽可能准确地捕获语义含义，但传递尽可能多的上下文。请注意，这就是 `ParentDocumentRetriever` 所做的。这里我们展示了底层的操作。

```python
# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="full_documents", embedding_function=OpenAIEmbeddings()
)
# 用于父文档的存储层
store = InMemoryByteStore()
id_key = "doc_id"
# 检索器（初始为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)
import uuid
doc_ids = [str(uuid.uuid4()) for _ in docs]
```

```python
# 用于创建较小块的分割器
child_text_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
```

```python
sub_docs = []
for i, doc in enumerate(docs):
    _id = doc_ids[i]
    _sub_docs = child_text_splitter.split_documents([doc])
    for _doc in _sub_docs:
        _doc.metadata[id_key] = _id
    sub_docs.extend(_sub_docs)
```

```python
retriever.vectorstore.add_documents(sub_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

```python
# 仅向量存储检索小块
retriever.vectorstore.similarity_search("justice breyer")[0]
```

```output
Document(page_content='Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.', metadata={'doc_id': '2fd77862-9ed5-4fad-bf76-e487b747b333', 'source': 'state_of_the_union.txt'})
```

```python
# 检索器返回较大块
len(retriever.get_relevant_documents("justice breyer")[0].page_content)
```

```output
9875
```

检索器在向量数据库上执行的默认搜索类型是相似性搜索。LangChain 向量存储还支持通过 [最大边际相关性](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.max_marginal_relevance_search) 进行搜索，因此，如果您希望使用这种方法，只需将 `search_type` 属性设置如下：

```python
from langchain.retrievers.multi_vector import SearchType
retriever.search_type = SearchType.mmr
len(retriever.get_relevant_documents("justice breyer")[0].page_content)
```

```output
9875
```

## 摘要

通常情况下，摘要可能更准确地概括一个块的内容，从而实现更好的检索效果。这里我们展示了如何创建摘要，然后嵌入它们。

```python
import uuid
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
```

```python
链 = (
    {"doc": lambda x: x.page_content}
    | ChatPromptTemplate.from_template("总结以下文档内容：\n\n{doc}")
    | ChatOpenAI(max_retries=0)
    | StrOutputParser()
)
```

```python
摘要 = chain.batch(docs, {"max_concurrency": 5})
```

```python
# 用于索引子块的向量存储
vectorstore = Chroma(collection_name="summaries", embedding_function=OpenAIEmbeddings())
# 用于存储父文档的存储层
store = InMemoryByteStore()
id_key = "doc_id"
# 检索器（初始为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)
doc_ids = [str(uuid.uuid4()) for _ in docs]
```

```python
摘要文档 = [
    Document(page_content=s, metadata={id_key: doc_ids[i]})
    for i, s in enumerate(summaries)
]
```

```python
retriever.vectorstore.add_documents(summary_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

```python
# # 如果需要，我们也可以将原始块添加到向量存储中
# for i, doc in enumerate(docs):
#     doc.metadata[id_key] = doc_ids[i]
# retriever.vectorstore.add_documents(docs)
```

```python
子文档 = vectorstore.similarity_search("justice breyer")
```

```python
sub_docs[0]
```

```output
Document(page_content="这份文件是拜登总统发表的讲话，涉及各种问题并概述了他对国家议程的看法。他强调了提名最高法院大法官的重要性，并介绍了他的提名人选凯坦吉·布朗·杰克逊法官。他强调了保护边境和改革移民制度的必要性，包括为梦想者和基本工作者提供公民身份途径。总统还讨论了保护妇女权利的重要性，包括获得医疗保健和选择权。他呼吁通过《平等法案》以保护 LGBTQ+ 权利。此外，拜登总统还讨论了解决阿片类药物滥用危机、改善心理健康服务、支持退伍军人和与癌症作斗争的必要性。他对美国的未来和美国人民的力量表示乐观。", metadata={'doc_id': '56345bff-3ead-418c-a4ff-dff203f77474'})
```

```python
检索到的文档 = retriever.get_relevant_documents("justice breyer")
```

```python
len(retrieved_docs[0].page_content)
```

```output
9194
```

## 假设性查询

LLM 还可用于生成可能针对特定文档提出的假设性问题列表。然后可以将这些问题嵌入其中

```python
functions = [
    {
        "name": "hypothetical_questions",
        "description": "生成假设性问题",
        "parameters": {
            "type": "object",
            "properties": {
                "questions": {
                    "type": "array",
                    "items": {"type": "string"},
                },
            },
            "required": ["questions"],
        },
    }
]
```

```python
from langchain_core.output_parsers.openai_functions import JsonKeyOutputFunctionsParser
chain = (
    {"doc": lambda x: x.page_content}
    # 只要求生成 3 个假设性问题，但可以调整
    | ChatPromptTemplate.from_template(
        "生成一个包含 3 个假设性问题的列表，这些问题可以用来回答下面的文档：\n\n{doc}"
    )
    | ChatOpenAI(max_retries=0, model="gpt-4").bind(
        functions=functions, function_call={"name": "hypothetical_questions"}
    )
    | JsonKeyOutputFunctionsParser(key_name="questions")
)
```

```python
chain.invoke(docs[0])
```

```output
["作者的第一次编程经历是什么样的？",
 '作者为什么在研究生阶段将注意力从 AI 转向 Lisp？',
 '是什么让作者考虑过从计算机科学转向艺术职业？']
```

```python
假设性问题 = chain.batch(docs, {"max_concurrency": 5})
```

```python
# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="hypo-questions", embedding_function=OpenAIEmbeddings()
)
# 用于存储父文档的存储层
store = InMemoryByteStore()
id_key = "doc_id"
# 检索器（初始为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=store,
    id_key=id_key,
)
doc_ids = [str(uuid.uuid4()) for _ in docs]
```

```python
问题文档 = []
for i, question_list in enumerate(hypothetical_questions):
    question_docs.extend(
        [Document(page_content=s, metadata={id_key: doc_ids[i]}) for s in question_list]
    )
```

```python
retriever.vectorstore.add_documents(question_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

```python
子文档 = vectorstore.similarity_search("justice breyer")
```

```python
sub_docs
```

```output
[文档(page_content='谁被提名为美国最高法院的法官？', metadata={'doc_id': '0b3a349e-c936-4e77-9c40-0a39fc3e07f0'}),
 文档(page_content='2010年，罗伯特·莫里斯向文档作者提出了什么建议？', metadata={'doc_id': 'b2b2cdca-988a-4af1-ba47-46170770bc8c'}),
 文档(page_content='个人情况如何影响决定放弃 Y Combinator 的领导权？', metadata={'doc_id': 'b2b2cdca-988a-4af1-ba47-46170770bc8c'}),
 文档(page_content='作者在1999年夏天离开雅虎的原因是什么？', metadata={'doc_id': 'ce4f4981-ca60-4f56-86f0-89466de62325'})]
```

```python
retrieved_docs = retriever.get_relevant_documents("布莱尔法官")
```

```python
len(retrieved_docs[0].page_content)
```

```output
9194
```