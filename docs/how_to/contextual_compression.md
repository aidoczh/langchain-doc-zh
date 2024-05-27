# 如何使用上下文压缩进行检索

检索的一个挑战是，当您将数据导入系统时，通常不知道文档存储系统将面临的具体查询。这意味着与查询相关的最重要信息可能埋藏在大量无关文本的文档中。将完整文档传递给应用程序可能导致更昂贵的LLM调用和更差的响应。

上下文压缩旨在解决这个问题。其思想很简单：不要立即返回检索到的文档，而是使用给定查询的上下文对其进行压缩，以便只返回相关信息。这里的“压缩”既指压缩单个文档的内容，也指整体过滤文档。

要使用上下文压缩检索器，您需要：

- 一个基本检索器

- 一个文档压缩器

上下文压缩检索器将查询传递给基本检索器，获取初始文档并将其通过文档压缩器。文档压缩器接受文档列表，并通过减少文档内容或完全删除文档来缩短列表。

## 入门

```python
# 用于打印文档的辅助函数
def pretty_print_docs(docs):
    print(
        f"\n{'-' * 100}\n".join(
            [f"Document {i+1}:\n\n" + d.page_content for i, d in enumerate(docs)]
        )
    )
```

## 使用普通向量存储检索器

让我们从初始化一个简单的向量存储检索器开始，并存储2023年的国情咨文（以块为单位）。我们可以看到，给定一个示例问题，我们的检索器返回了一个或两个相关文档和几个无关文档。即使相关文档中也有很多无关信息。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
documents = TextLoader("state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)
retriever = FAISS.from_documents(texts, OpenAIEmbeddings()).as_retriever()
docs = retriever.invoke("What did the president say about Ketanji Brown Jackson")
pretty_print_docs(docs)
```

```output
Document 1:
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
----------------------------------------------------------------------------------------------------
Document 2:
A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. 
And if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. 
We can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  
We’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  
We’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. 
We’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.
----------------------------------------------------------------------------------------------------
Document 3:
And for our LGBTQ+ Americans, let’s finally get the bipartisan Equality Act to my desk. The onslaught of state laws targeting transgender Americans and their families is wrong. 
As I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. 
While it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice. 
And soon, we’ll strengthen the Violence Against Women Act that I first wrote three decades ago. It is important for us to show the nation that we can come together and do big things. 
So tonight I’m offering a Unity Agenda for the Nation. Four big things we can do together.  
First, beat the opioid epidemic.
----------------------------------------------------------------------------------------------------
Document 4:
Tonight, I’m announcing a crackdown on these companies overcharging American businesses and consumers. 
And as Wall Street firms take over more nursing homes, quality in those homes has gone down and costs have gone up.  
That ends on my watch. 
Medicare is going to set higher standards for nursing homes and make sure your loved ones get the care they deserve and expect. 
We’ll also cut costs and keep the economy going strong by giving workers a fair shot, provide more training and apprenticeships, hire them based on their skills not degrees. 
Let’s pass the Paycheck Fairness Act and paid leave.  
Raise the minimum wage to $15 an hour and extend the Child Tax Credit, so no one has to raise a family in poverty. 
Let’s increase Pell Grants and increase our historic support of HBCUs, and invest in what Jill—our First Lady who teaches full-time—calls America’s best-kept secret: community colleges.
```

## 使用 `LLMChainExtractor` 添加上下文压缩

现在让我们用 `ContextualCompressionRetriever` 包装我们的基础检索器。我们将添加一个 `LLMChainExtractor`，它将遍历最初返回的文档，并从每个文档中提取与查询相关的内容。

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
compressor = LLMChainExtractor.from_llm(llm)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "总统对Ketanji Jackson Brown说了什么"
)
pretty_print_docs(compressed_docs)
```

```output
文档 1:
我是在4天前提名了巡回上诉法院法官Ketanji Brown Jackson时做的。
```

## 更多内置压缩器：过滤器

### `LLMChainFilter`

`LLMChainFilter` 是一个稍微简单但更健壮的压缩器，它使用 LLM 链来决定最初检索到的文档中要过滤掉哪些，要返回哪些，而不会改变文档内容。

```python
from langchain.retrievers.document_compressors import LLMChainFilter
_filter = LLMChainFilter.from_llm(llm)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=_filter, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "总统对Ketanji Jackson Brown说了什么"
)
pretty_print_docs(compressed_docs)
```

```output
文档 1:
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法史蒂芬·布雷耶——一位陆军退伍军人，宪法学者，美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
我是在4天前提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律专家，将继续布雷耶法官的卓越传统。
```

### `EmbeddingsFilter`

对每个检索到的文档进行额外的 LLM 调用是昂贵且缓慢的。`EmbeddingsFilter` 提供了一种更便宜、更快速的选项，它通过嵌入文档和查询，只返回那些与查询具有足够相似嵌入的文档。

```python
from langchain.retrievers.document_compressors import EmbeddingsFilter
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
embeddings_filter = EmbeddingsFilter(embeddings=embeddings, similarity_threshold=0.76)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=embeddings_filter, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "总统对Ketanji Jackson Brown说了什么"
)
pretty_print_docs(compressed_docs)
```

```output
文档 1:
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法史蒂芬·布雷耶——一位陆军退伍军人，宪法学者，美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
我是在4天前提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律专家，将继续布雷耶法官的卓越传统。
----------------------------------------------------------------------------------------------------
文档 2:
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个共识建设者。自提名以来，她得到了广泛的支持——从警察工会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们两者都可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测走私毒品。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
```

我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。

我们正在确保南美和中美的合作伙伴承诺接纳更多难民并保护自己的边境。

## 将压缩器和文档转换器串联在一起

使用 `DocumentCompressorPipeline`，我们还可以轻松地将多个压缩器按顺序组合在一起。除了压缩器，我们还可以将 `BaseDocumentTransformer` 添加到我们的流水线中，它们不执行任何上下文压缩，只是对一组文档进行一些转换。例如，`TextSplitter` 可以用作文档转换器，将文档分割成较小的片段，而 `EmbeddingsRedundantFilter` 可以根据文档之间的嵌入相似性过滤掉冗余文档。

下面我们通过首先将文档分割成较小的块，然后删除冗余文档，最后根据与查询相关性进行过滤来创建一个压缩器流水线。

```python
from langchain.retrievers.document_compressors import DocumentCompressorPipeline
from langchain_community.document_transformers import EmbeddingsRedundantFilter
from langchain_text_splitters import CharacterTextSplitter
splitter = CharacterTextSplitter(chunk_size=300, chunk_overlap=0, separator=". ")
redundant_filter = EmbeddingsRedundantFilter(embeddings=embeddings)
relevant_filter = EmbeddingsFilter(embeddings=embeddings, similarity_threshold=0.76)
pipeline_compressor = DocumentCompressorPipeline(
    transformers=[splitter, redundant_filter, relevant_filter]
)
```

```python
compression_retriever = ContextualCompressionRetriever(
    base_compressor=pipeline_compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "What did the president say about Ketanji Jackson Brown"
)
pretty_print_docs(compressed_docs)
```

```output
文档 1：
作为总统，最重要的宪法责任之一就是提名某人担任美国最高法院的职位。
4天前，我提名了巡回上诉法院法官Ketanji Brown Jackson。
----------------------------------------------------------------------------------------------------
文档 2：
正如我去年所说的，特别是对我们年轻的跨性别美国人，作为你们的总统，我将永远支持你们，让你们可以做自己，发挥上帝赋予你们的潜力。
尽管我们似乎从不达成一致，但事实并非如此。去年，我签署了80项两党议案成为法律。
----------------------------------------------------------------------------------------------------
文档 3：
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个共识的建设者。
----------------------------------------------------------------------------------------------------
文档 4：
自从她被提名以来，她得到了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民系统。
我们可以两者兼顾。
```