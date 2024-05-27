

# 如何使用父文档检索器

在拆分文档以进行检索时，通常存在着相互冲突的愿望：

1. 您可能希望拥有小型文档，以便它们的嵌入可以最准确地反映它们的含义。如果太长，那么嵌入可能会失去意义。

2. 您希望拥有足够长的文档，以保留每个块的上下文。

`ParentDocumentRetriever` 通过拆分和存储小块数据来实现这种平衡。在检索过程中，它首先获取小块，然后查找这些块的父 ID，并返回那些更大的文档。

请注意，“父文档”指的是小块来源的文档。这可以是整个原始文档，也可以是较大的块。

```python
from langchain.retrievers import ParentDocumentRetriever
```
```python
from langchain.storage import InMemoryStore
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```
```python
loaders = [
    TextLoader("paul_graham_essay.txt"),
    TextLoader("state_of_the_union.txt"),
]
docs = []
for loader in loaders:
    docs.extend(loader.load())
```

## 检索完整文档

在此模式下，我们希望检索完整文档。因此，我们只指定一个子拆分器。

```python
# 此文本拆分器用于创建子文档
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="full_documents", embedding_function=OpenAIEmbeddings()
)
# 父文档的存储层
store = InMemoryStore()
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
)
```
```python
retriever.add_documents(docs, ids=None)
```

这应该会产生两个键，因为我们添加了两个文档。

```python
list(store.yield_keys())
```
```output
['9a63376c-58cc-42c9-b0f7-61f0e1a3a688',
 '40091598-e918-4a18-9be0-f46413a95ae4']
```

现在让我们调用向量存储的搜索功能 - 我们应该看到它返回小块（因为我们存储了小块）。

```python
sub_docs = vectorstore.similarity_search("justice breyer")
```
```python
print(sub_docs[0].page_content)
```
```output
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院法官。
```

现在让我们从整体检索器中检索。这应该返回大文档 - 因为它返回了包含小块的文档。

```python
retrieved_docs = retriever.invoke("justice breyer")
```
```python
len(retrieved_docs[0].page_content)
```
```output
38540
```

## 检索较大块

有时，完整文档可能太大，我们不希望按原样检索它们。在这种情况下，我们真正想要做的是首先将原始文档拆分为较大块，然后将其拆分为较小块。然后我们索引较小块，但在检索时检索较大块（但仍不是完整文档）。

```python
# 此文本拆分器用于创建父文档
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)
# 此文本拆分器用于创建子文档
# 它应创建比父文档小的文档
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="split_parents", embedding_function=OpenAIEmbeddings()
)
# 父文档的存储层
store = InMemoryStore()
```
```python
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)
```
```python
retriever.add_documents(docs)
```

现在我们可以看到不止两个文档了 - 这些是较大的块。

```python
len(list(store.yield_keys()))
```
```output
66
```

让我们确保底层向量存储仍然检索小块。

```python
sub_docs = vectorstore.similarity_search("justice breyer")
```
```python
print(sub_docs[0].page_content)
```
```output
今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院法官。
```python
retrieved_docs = retriever.invoke("justice breyer")
```
```python
len(retrieved_docs[0].page_content)
```
```output

1849

```
```python
print(retrieved_docs[0].page_content)
```
```output

在一个又一个州，新的法律不仅被通过来压制选民权利，还被用来颠覆整个选举过程。

我们不能让这种情况发生。

今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。而且，趁机通过《披露法案》，这样美国人就能知道谁在资助我们的选举。

今晚，我想向一个致力于为国家服务的人致敬：司法部长斯蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。

总统最重要的宪法责任之一就是提名人选担任美国最高法院法官。

而我在4天前就提名了联邦上诉法院法官凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）。她是我们国家最顶尖的法律智慧之一，将继续布雷耶法官的卓越传统。

她曾是一位顶级的私人执业律师，也曾是一位联邦公共辩护人。她来自一家公立学校教育工作者和警察的家庭。她是一个能够凝聚共识的人。自从她被提名以来，她得到了广泛的支持，包括警察兄弟会和民主党和共和党任命的前法官。

如果我们要推进自由和正义，我们需要保护边境并修复移民系统。

我们可以两者兼顾。在我们的边境，我们安装了先进的扫描仪等新技术，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩运者。

我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。

我们正在与南美和中美洲的伙伴建立承诺，以接纳更多的难民并保护他们的边境。

```
