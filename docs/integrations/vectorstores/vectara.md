# Vectara

[Vectara](https://vectara.com/) 是一款可信赖的 GenAI 平台，提供了一个易于使用的 API，用于文档索引和查询。

Vectara 提供了用于检索增强生成（Retrieval Augmented Generation，[RAG](https://vectara.com/grounded-generation/)）的端到端托管服务，其中包括：

1. 从文档文件中提取文本并将其分块成句子的方法。

2. 最先进的 [Boomerang](https://vectara.com/how-boomerang-takes-retrieval-augmented-generation-to-the-next-level-via-grounded-generation/) 嵌入模型。每个文本块都使用 Boomerang 编码为向量嵌入，并存储在 Vectara 内部知识（向量+文本）存储中。

3. 一个查询服务，自动将查询编码为嵌入，并检索最相关的文本段落（包括对 [Hybrid Search](https://docs.vectara.com/docs/api-reference/search-apis/lexical-matching) 和 [MMR](https://vectara.com/get-diverse-results-and-comprehensive-summaries-with-vectaras-mmr-reranker/) 的支持）。

4. 一个选项，可以基于检索到的文档创建 [生成摘要](https://docs.vectara.com/docs/learn/grounded-generation/grounded-generation-overview)，包括引用。

查看 [Vectara API 文档](https://docs.vectara.com/docs/) 以获取有关如何使用 API 的更多信息。

本笔记本展示了如何在使用 Vectara 作为向量存储（不包括摘要）时使用基本的检索功能，包括：`similarity_search` 和 `similarity_search_with_score`，以及使用 LangChain 的 `as_retriever` 功能。

# 设置

使用 Vectara 与 LangChain 需要 Vectara 账户。要开始使用，请按照以下步骤操作：

1. 如果尚未拥有 Vectara 账户，请 [注册](https://www.vectara.com/integrations/langchain) 一个。完成注册后，您将获得一个 Vectara 客户 ID。您可以通过单击 Vectara 控制台窗口右上角的您的名称来查找您的客户 ID。

2. 在您的账户中，您可以创建一个或多个语料库。每个语料库代表一个区域，用于存储来自输入文档的文本数据。要创建语料库，请使用 **"Create Corpus"** 按钮。然后，为您的语料库提供一个名称和描述。您还可以选择定义过滤属性并应用一些高级选项。如果单击您创建的语料库，您可以在顶部看到其名称和语料库 ID。

3. 接下来，您需要创建 API 密钥以访问语料库。在语料库视图中，单击 **"Authorization"** 选项卡，然后单击 **"Create API Key"** 按钮。为您的密钥命名，并选择您希望为密钥进行查询还是查询+索引。单击 **"Create"**，现在您有一个活动的 API 密钥。请保密此密钥。

要使用 LangChain 与 Vectara，您需要这三个值：客户 ID、语料库 ID 和 API 密钥。

您可以通过以下两种方式将这些值提供给 LangChain：

1. 在您的环境中包含这三个变量：`VECTARA_CUSTOMER_ID`、`VECTARA_CORPUS_ID` 和 `VECTARA_API_KEY`。

   > 例如，您可以使用 os.environ 和 getpass 设置这些变量，如下所示：

   ```python
   import os
   import getpass
   os.environ["VECTARA_CUSTOMER_ID"] = getpass.getpass("Vectara Customer ID:")
   os.environ["VECTARA_CORPUS_ID"] = getpass.getpass("Vectara Corpus ID:")
   os.environ["VECTARA_API_KEY"] = getpass.getpass("Vectara API Key:")
   ```

2. 将它们添加到 Vectara vectorstore 构造函数：

   ```python
   vectorstore = Vectara(
                   vectara_customer_id=vectara_customer_id,
                   vectara_corpus_id=vectara_corpus_id,
                   vectara_api_key=vectara_api_key
               )
   ```

## 从 LangChain 连接到 Vectara

要开始，让我们使用 from_documents() 方法摄取文档。

在这里，我们假设您已将您的 VECTARA_CUSTOMER_ID、VECTARA_CORPUS_ID 和查询+索引 VECTARA_API_KEY 添加为环境变量。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import Vectara
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
vectara = Vectara.from_documents(
    docs,
    embedding=FakeEmbeddings(size=768),
    doc_metadata={"speech": "state-of-the-union"},
)
```

Vectara 的索引 API 提供了一个文件上传 API，其中文件直接由 Vectara 处理 - 预处理、最佳分块并添加到 Vectara 向量存储中。

为了使用这个功能，我们添加了 add_files() 方法（以及 from_files()）。

让我们看看它的运行情况。我们选择了两个要上传的 PDF 文档：

1. "我有一个梦" 演讲，由金博士。

## 丘吉尔的“我们将在海滩上作战”演讲

```python
import tempfile
import urllib.request
urls = [
    [
        "https://www.gilderlehrman.org/sites/default/files/inline-pdfs/king.dreamspeech.excerpts.pdf",
        "I-have-a-dream",
    ],
    [
        "https://www.parkwayschools.net/cms/lib/MO01931486/Centricity/Domain/1578/Churchill_Beaches_Speech.pdf",
        "我们将在海滩上作战",
    ],
]
files_list = []
for url, _ in urls:
    name = tempfile.NamedTemporaryFile().name
    urllib.request.urlretrieve(url, name)
    files_list.append(name)
docsearch: Vectara = Vectara.from_files(
    files=files_list,
    embedding=FakeEmbeddings(size=768),
    metadatas=[{"url": url, "speech": title} for url, title in urls],
)
```

## 相似度搜索

使用 Vectara 最简单的场景是执行相似度搜索。

```python
query = "总统对凯坦吉·布朗·杰克逊有何评价"
found_docs = vectara.similarity_search(
    query, n_sentence_context=0, filter="doc.speech = 'state-of-the-union'"
)
```

```python
found_docs
```

```output
[Document(page_content='我在4天前这样做了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '596', 'len': '97', 'speech': 'state-of-the-union'}),
 Document(page_content='在这场斗争中，正如泽连斯基总统在他向欧洲议会的演讲中所说的：“光明将战胜黑暗。”', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '141', 'len': '117', 'speech': 'state-of-the-union'}),
 Document(page_content='正如俄亥俄州参议员谢罗德·布朗所说：“是时候埋葬‘锈带’这个标签了。”', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '0', 'len': '77', 'speech': 'state-of-the-union'}),
 Document(page_content='上个月，我宣布了我们计划加速执行奥巴马总统六年前要求我领导的癌症登月计划。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '0', 'len': '122', 'speech': 'state-of-the-union'}),
 Document(page_content='他认为他可以入侵乌克兰，世界会屈服于他。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '664', 'len': '68', 'speech': 'state-of-the-union'}),
 Document(page_content='这就是为什么我担任总统后做的第一件事之一是努力通过《美国拯救计划》。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '314', 'len': '97', 'speech': 'state-of-the-union'}),
 Document(page_content='他认为他可以在国内分裂我们。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '160', 'len': '42', 'speech': 'state-of-the-union'}),
 Document(page_content='他见识到了乌克兰人民。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '788', 'len': '28', 'speech': 'state-of-the-union'}),
 Document(page_content='他认为西方和北约不会作出回应。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '113', 'len': '46', 'speech': 'state-of-the-union'}),
 Document(page_content='在这座国会大厦里，一代又一代的美国人在巨大的争议中辩论重大问题，并做出了伟大的事业。', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '772', 'len': '131', 'speech': 'state-of-the-union'})]
```

```python
print(found_docs[0].page_content)
```

```output
我在4天前这样做了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。
```

## 带分数的相似度搜索

有时我们可能希望进行搜索，同时获得一个相关性分数，以了解特定结果的好坏程度。

```python
query = "总统对凯坦吉·布朗·杰克逊有何评价"
found_docs = vectara.similarity_search_with_score(
    query,
    filter="doc.speech = 'state-of-the-union'",
    score_threshold=0.2,
)
```

```python
document, score = found_docs[0]
print(document.page_content)
print(f"\n分数: {score}")
```

```output
贝雷尔法官，感谢您的服务。总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。我在4天前这样做了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续贝雷尔法官的卓越传统。一位前私人执业的顶尖诉讼律师。
分数: 0.74179757
```

现在让我们为我们上传的文件内容进行相似搜索

```python
query = "我们必须永远进行我们的斗争"
最低分数 = 1.2
found_docs = vectara.similarity_search_with_score(
    query,
    filter="doc.speech = 'I-have-a-dream'",
    score_threshold=最低分数,
)
print(f"在这个阈值为 {最低分数} 时，我们有 {len(found_docs)} 个文档")
```

```output
在这个阈值为 1.2 时，我们有 0 个文档
```

```python
query = "我们必须永远进行我们的斗争"
最低分数 = 0.2
found_docs = vectara.similarity_search_with_score(
    query,
    filter="doc.speech = 'I-have-a-dream'",
    score_threshold=最低分数,
)
print(f"在这个阈值为 {最低分数} 时，我们有 {len(found_docs)} 个文档")
使用阈值为0.2，我们有10个文档。
MMR是许多应用程序的重要检索功能，通过重新排列搜索结果来改善结果的多样性，从而提高GenAI应用程序的搜索效果。
让我们看看Vectara是如何工作的：
```python

query = "经济状况"

found_docs = vectara.similarity_search(

    query,

    n_sentence_context=0,

    filter="doc.speech = '国情咨文'",

    k=5,

    mmr_config={"is_enabled": True, "mmr_k": 50, "diversity_bias": 0.0},

)

print("\n\n".join([x.page_content for x in found_docs]))

```
输出结果为：
```

经济援助。

发展劳动力。从底层和中间层开始建设经济，而不是从顶层开始。

当我们投资于我们的工人，当我们共同从底层和中间层开始建设经济时，我们可以做到长时间以来没有做到的事情：建设一个更美好的美国。

去年，我们的经济增长率达到了5.7%，是近40年来增长最强劲的一年，这是将根本性变革带给一个长期以来对这个国家的劳动人民没有起作用的经济的第一步。

经济学家称之为“增加我们经济的生产能力”。

```python
query = "经济状况"
found_docs = vectara.similarity_search(
    query,
    n_sentence_context=0,
    filter="doc.speech = '国情咨文'",
    k=5,
    mmr_config={"is_enabled": True, "mmr_k": 50, "diversity_bias": 1.0},
)
print("\n\n".join([x.page_content for x in found_docs]))
```
输出结果为：
```
经济援助。
俄罗斯股市已经贬值了40%，交易仍然暂停。
但是这种滴灌理论导致了经济增长的疲软，工资水平下降，赤字增加，以及近一个世纪以来富人与其他人之间的差距最大。
在一个又一个州，已经通过了新的法律，不仅压制选民权利，而且颠覆整个选举过程。
联邦政府每年花费约6000亿美元来确保国家的安全。
```

正如您所看到的，第一个示例中diversity_bias被设置为0.0（等同于禁用多样性重新排序），结果是最相关的前5个文档。通过设置diversity_bias=1.0，我们最大化了多样性，可以看到结果中的前几个文档在语义意义上更加多样化。

## Vectara作为检索器

最后，让我们看看如何使用Vectara的`as_retriever()`接口：

```python
retriever = vectara.as_retriever()
retriever
```
输出结果为：
```
VectorStoreRetriever(tags=['Vectara'], vectorstore=<langchain_community.vectorstores.vectara.Vectara object at 0x109a3c760>)
```

```python
query = "总统对Ketanji Brown Jackson有什么说法"
retriever.invoke(query)[0]
```
输出结果为：
```
Document(page_content='Justice Breyer, thank you for your service. One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence. A former top litigator in private practice.', metadata={'source': 'langchain', 'lang': 'eng', 'offset': '596', 'len': '97', 'speech': 'state-of-the-union'})
```