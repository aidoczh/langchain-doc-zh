# Weaviate

本文介绍了如何使用 `langchain-weaviate` 包在 LangChain 中开始使用 Weaviate 向量存储。

> [Weaviate](https://weaviate.io/) 是一个开源的向量数据库。它允许您存储来自您喜爱的机器学习模型的数据对象和向量嵌入，并能够无缝地扩展到数十亿个数据对象。

要使用此集成，您需要运行一个 Weaviate 数据库实例。

## 最低版本

此模块需要 Weaviate `1.23.7` 或更高版本。但是，我们建议您使用最新版本的 Weaviate。

## 连接到 Weaviate

在本文中，我们假设您在 `http://localhost:8080` 上运行了一个本地的 Weaviate 实例，并且端口 50051 用于 [gRPC 通信](https://weaviate.io/blog/grpc-performance-improvements)。因此，我们将使用以下代码连接到 Weaviate：

```python
weaviate_client = weaviate.connect_to_local()
```

### 其他部署选项

Weaviate 可以以许多不同的方式进行[部署](https://weaviate.io/developers/weaviate/starter-guides/which-weaviate)，例如使用[Weaviate Cloud Services (WCS)](https://console.weaviate.cloud)、[Docker](https://weaviate.io/developers/weaviate/installation/docker-compose)或[Kubernetes](https://weaviate.io/developers/weaviate/installation/kubernetes)。

如果您的 Weaviate 实例以其他方式部署，可以在[此处阅读更多信息](https://weaviate.io/developers/weaviate/client-libraries/python#instantiate-a-client)关于连接到 Weaviate 的不同方式。您可以使用不同的[辅助函数](https://weaviate.io/developers/weaviate/client-libraries/python#python-client-v4-helper-functions)，或者[创建一个自定义实例](https://weaviate.io/developers/weaviate/client-libraries/python#python-client-v4-explicit-connection)。

> 请注意，您需要一个 `v4` 客户端 API，它将创建一个 `weaviate.WeaviateClient` 对象。

### 认证

一些 Weaviate 实例，例如在 WCS 上运行的实例，启用了认证，例如 API 密钥和/或用户名+密码认证。

阅读[客户端认证指南](https://weaviate.io/developers/weaviate/client-libraries/python#authentication)以获取更多信息，以及[深入的认证配置页面](https://weaviate.io/developers/weaviate/configuration/authentication)。

## 安装

```python
# 安装包
# %pip install -Uqq langchain-weaviate
# %pip install openai tiktoken langchain
```

## 环境设置

本文使用 `OpenAIEmbeddings` 通过 OpenAI API。我们建议获取一个 OpenAI API 密钥，并将其作为名为 `OPENAI_API_KEY` 的环境变量导出。

完成后，您的 OpenAI API 密钥将被自动读取。如果您对环境变量不熟悉，可以在[此处](https://docs.python.org/3/library/os.html#os.environ)或[此指南](https://www.twilio.com/en-us/blog/environment-variables-python)中阅读更多关于它们的信息。

# 用法

## 通过相似性查找对象

以下是一个示例，演示如何通过查询查找与之相似的对象，从数据导入到查询 Weaviate 实例。

### 步骤 1：数据导入

首先，我们将创建要添加到 `Weaviate` 的数据，方法是加载并分块长文本文件的内容。

```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

现在，我们可以导入数据。

要这样做，连接到 Weaviate 实例，并使用生成的 `weaviate_client` 对象。例如，我们可以将文档导入如下所示：

```python
import weaviate
from langchain_weaviate.vectorstores import WeaviateVectorStore
```

```python
weaviate_client = weaviate.connect_to_local()
db = WeaviateVectorStore.from_documents(docs, embeddings, client=weaviate_client)
```

### 第二步：执行搜索

现在我们可以执行相似度搜索。这将返回与查询文本最相似的文档，基于存储在 Weaviate 中的嵌入和从查询文本生成的等效嵌入。

```python
query = "总统对Ketanji Brown Jackson说了什么"
docs = db.similarity_search(query)
# 打印每个结果的前100个字符
for i, doc in enumerate(docs):
    print(f"\n文档 {i+1}:")
    print(doc.page_content[:100] + "...")
```

```output
文档 1:
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》...
文档 2:
许多家庭都是靠着发薪支票度日，努力跟上不断上涨的生活成本...
文档 3:
副总统哈里斯和我一起竞选，为美国提出了新的经济愿景。
投资于美国...
文档 4:
一位曾在私人执业中担任高级诉讼律师的人。一位前联邦公共辩护律师。来自一家...
```

您还可以添加过滤器，根据过滤条件包含或排除结果。 (查看[更多过滤器示例](https://weaviate.io/developers/weaviate/search/filters)。)

```python
from weaviate.classes.query import Filter
for filter_str in ["blah.txt", "state_of_the_union.txt"]:
    search_filter = Filter.by_property("source").equal(filter_str)
    filtered_search_results = db.similarity_search(query, filters=search_filter)
    print(len(filtered_search_results))
    if filter_str == "state_of_the_union.txt":
        assert len(filtered_search_results) > 0  # 应至少有一个结果
    else:
        assert len(filtered_search_results) == 0  # 不应有结果
```

```output
0
4
```

还可以提供 `k`，它是要返回的结果数量的上限。

```python
search_filter = Filter.by_property("source").equal("state_of_the_union.txt")
filtered_search_results = db.similarity_search(query, filters=search_filter, k=3)
assert len(filtered_search_results) <= 3
```

### 量化结果相似性

您可以选择检索相关性“分数”。这是一个相对分数，表示特定搜索结果在搜索结果池中的好坏程度。

请注意，这是相对分数，意味着不应用于确定相关性的阈值。但是，它可用于比较整个搜索结果集中不同搜索结果的相关性。

```python
docs = db.similarity_search_with_score("country", k=5)
for doc in docs:
    print(f"{doc[1]:.3f}", ":", doc[0].page_content[:100] + "...")
```

```output
0.935 : 为此，我们动员了美国地面部队、空中中队和舰队部署来保护...
0.500 : 并建立了世界上最强大、最自由、最繁荣的国家。
现在是...
0.462 : 如果您从俄亥俄州哥伦布向东行驶20英里，您会发现1000英亩的空地。
看起来...
0.450 : 我的报告是：联邦政府的状况良好——因为您，美国人民，是强大的...
0.442 : 今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》...
```

## 搜索机制

`similarity_search` 使用 Weaviate 的[混合搜索](https://weaviate.io/developers/weaviate/api/graphql/search-operators#hybrid)。

混合搜索结合了向量搜索和关键字搜索，`alpha` 是向量搜索的权重。`similarity_search` 函数允许您将额外的参数作为 kwargs 传递。查看此[参考文档](https://weaviate.io/developers/weaviate/api/graphql/search-operators#hybrid)以获取可用参数。

因此，您可以通过添加 `alpha=0` 执行纯关键字搜索，如下所示：

```python
docs = db.similarity_search(query, alpha=0)
docs[0]
```

```output
Document(page_content='今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》。在此期间，通过《披露法案》，让美国人民知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。司法部长布雷耶，感谢您的服务。\n\n总统拥有的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。\n\n4天前，我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续布雷耶法官的卓越传统。', metadata={'source': 'state_of_the_union.txt'})
```

## 持久性

通过 `langchain-weaviate` 添加的任何数据将根据其配置在 Weaviate 中持久存在。

例如，WCS 实例配置为无限期持久化数据，Docker 实例可以设置为在卷中持久化数据。了解更多关于[Weaviate 的持久性](https://weaviate.io/developers/weaviate/configuration/persistence)。

## 多租户

[多租户](https://weaviate.io/developers/weaviate/concepts/data#multi-tenancy)允许您在单个 Weaviate 实例中拥有大量隔离的数据集合，这些数据集合具有相同的集合配置。这对于多用户环境非常有用，比如构建 SaaS 应用程序，其中每个最终用户将拥有自己的隔离数据集合。

要使用多租户，向量存储需要了解 `tenant` 参数。

因此，在添加任何数据时，请提供如下所示的 `tenant` 参数。

```python
db_with_mt = WeaviateVectorStore.from_documents(
    docs, embeddings, client=weaviate_client, tenant="Foo"
)
```

```output
2024年3月26日 下午3:40 - langchain_weaviate.vectorstores - 信息 - 索引 LangChain_30b9273d43b3492db4fb2aba2e0d6871 中不存在租户 Foo。正在创建租户。
```

在执行查询时，还需要提供 `tenant` 参数。

```python
db_with_mt.similarity_search(query, tenant="Foo")
```

```output
[Document(page_content='今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就能知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名人选担任美国最高法院法官。\n\n而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶司法部长的卓越传统。', metadata={'source': 'state_of_the_union.txt'}),
 Document(page_content='许多家庭都是靠着每个月的薪水度日，艰难地应对食品、汽油、住房等成本的上涨。\n\n我理解。 \n\n我记得我爸爸曾经不得不离开我们在宾夕法尼亚州斯克兰顿的家去找工作。我在一个家庭长大，如果食品价格上涨，你会感受到它。\n\n这就是为什么我担任总统后做的第一件事之一是努力通过《美国拯救计划》。\n\n因为人们正在受苦。我们需要行动，而我们也做到了。\n\n在我们历史上的一个关键时刻，几乎没有哪项立法比这个更多地帮助我们摆脱危机。\n\n它推动了我们的努力为全国人民接种疫苗，对抗COVID-19。\n\n它为数千万美国人提供了即时的经济援助。\n\n帮助他们解决温饱问题，保障他们的住房，并降低了健康保险的费用。\n\n正如我爸爸过去常说的那样，它给了人们一点喘息的时间。', metadata={'source': 'state_of_the_union.txt'}),
 Document(page_content='他和他爸爸都患有1型糖尿病，这意味着他们每天都需要胰岛素。\n\n制造一瓶胰岛素的成本约为10美元。\n\n但制药公司向像约书亚和他爸爸这样的家庭收取的价格高达原价的30倍。我曾与约书亚的妈妈交谈过。\n\n想象一下，看着你需要胰岛素的孩子，却不知道如何支付它的费用。\n\n这对你的尊严、你与孩子对视的能力、以及你期望成为的父母形象都有何影响。\n\n约书亚今晚和我们在一起。昨天是他的生日。生日快乐，伙计。\n\n为了约书亚，以及其他20万名患有1型糖尿病的年轻人，让我们将胰岛素的费用限制在每月35美元，这样每个人都能负担得起。\n\n制药公司仍然会过得很好。而且顺便说一句，让医疗保险谈判降低处方药的价格，就像退伍军人事务部已经做的那样。', metadata={'source': 'state_of_the_union.txt'}),
 Document(page_content='普京最新对乌克兰的攻击是经过预谋的、无端的。\n\n他拒绝了反复的外交努力。\n\n他认为西方和北约不会作出回应。他认为他可以在国内分裂我们。普京错了。我们已经准备好了。以下是我们所做的。\n\n我们进行了充分和谨慎的准备。\n\n我们花了数月时间建立了一个由来自欧洲、美洲、亚洲和非洲的其他热爱自由的国家组成的联盟，以对抗普京。\n\n我花了无数个小时团结我们的欧洲盟友。我们提前向全世界分享了我们所知道的普京的计划，以及他将如何试图虚假地为自己的侵略行为辩护。\n\n我们用真理反驳了俄罗斯的谎言。\n\n现在他采取了行动，自由世界正在追究他的责任。\n\n与欧盟的27个成员国一起，包括法国、德国、意大利，以及英国、加拿大、日本、韩国、澳大利亚、新西兰和许多其他国家，甚至还有瑞士。', metadata={'source': 'state_of_the_union.txt'})]
```

## 检索器选项

Weaviate 还可以用作检索器

### 最大边际相关搜索（MMR）

除了在检索器对象中使用 `similaritysearch`，您还可以使用 `mmr`。

```python
retriever = db.as_retriever(search_type="mmr")
retriever.invoke(query)[0]
```

```output
/workspaces/langchain-weaviate/.venv/lib/python3.12/site-packages/pydantic/main.py:1024: PydanticDeprecatedSince20: `dict` 方法已被弃用；请改用 `model_dump`。在 Pydantic V2.0 中已弃用，将在 V3.0 中移除。请参阅 Pydantic V2 迁移指南 https://errors.pydantic.dev/2.6/migration/
  warnings.warn('The `dict` method is deprecated; use `model_dump` instead.', category=PydanticDeprecatedSince20)
```

今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法》。顺便一提，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院的退休法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前就提名了联邦上诉法院法官凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）。她是我们国家顶尖的法律专家之一，将继续延续布雷耶法官的卓越传统。 [20]

![图片](https://example.com/image.jpg)

---

使用 LangChain

大型语言模型（LLM）的一个已知限制是，它们的训练数据可能过时，或者不包括您所需的特定领域知识。

看下面的例子：

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
llm.predict("What did the president say about Justice Breyer")
```

抱歉，我无法提供实时信息，因为我的回答是基于许可数据、人类训练者创建的数据和公开可用数据的混合生成的。最后更新是在2021年10月。

---

向量存储通过提供一种存储和检索相关信息的方式来补充LLM，这使您可以结合LLM的推理和语言能力与向量存储的检索相关信息的能力。

结合LLM和向量存储的两个著名应用是：

- 问答

- 检索增强生成（RAG）

### 带来源的问答

在 LangChain 中，通过使用向量存储，可以增强问答功能。让我们看看如何实现这一点。

这部分使用 `RetrievalQAWithSourcesChain`，它从索引中查找文档。

首先，我们将再次对文本进行分块，并将其导入到 Weaviate 向量存储中。

```python
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_openai import OpenAI
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_text(state_of_the_union)
```

```python
docsearch = WeaviateVectorStore.from_texts(
    texts,
    embeddings,
    client=weaviate_client,
    metadatas=[{"source": f"{i}-pl"} for i in range(len(texts))],
)
```

现在我们可以构建链，指定检索器：

```python
chain = RetrievalQAWithSourcesChain.from_chain_type(
    OpenAI(temperature=0), chain_type="stuff", retriever=docsearch.as_retriever()
)
```

然后运行链，提出问题：

```python
chain(
    {"question": "What did the president say about Justice Breyer"},
    return_only_outputs=True,
)
```

### 检索增强生成

另一个将LLM和向量存储结合的非常流行的应用是检索增强生成（Retrieval-Augmented Generation，RAG）。这是一种使用检索器从向量存储中找到相关信息，然后使用LLM根据检索到的数据和提示生成输出的技术。

我们从类似的设置开始：

```python
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_text(state_of_the_union)
```

```python
docsearch = WeaviateVectorStore.from_texts(
    texts,
    embeddings,
    client=weaviate_client,
    metadatas=[{"source": f"{i}-pl"} for i in range(len(texts))],
)
retriever = docsearch.as_retriever()
```

我们需要构建一个RAG模型的模板，以便检索到的信息可以填充到模板中。

```python
from langchain_core.prompts import ChatPromptTemplate
template = """You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer:
"""
prompt = ChatPromptTemplate.from_template(template)
print(prompt)
```

```python
input_variables=['context', 'question'] messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['context', 'question'], template="You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.\nQuestion: {question}\nContext: {context}\nAnswer:\n"))]
```

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
```

运行这个代码块，我们会得到一个非常相似的输出。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
rag_chain.invoke("What did the president say about Justice Breyer")
```

```output

总统表彰了史蒂芬·布雷耶法官，以表彰他作为陆军退伍军人、宪法学者和美国最高法院即将退休法官对国家的服务。总统还提到提名联邦上诉法院法官凯坦吉·布朗·杰克逊继续延续布雷耶法官的卓越传统。总统对布雷耶法官表示感激，并强调提名某人担任美国最高法院的重要性。

### 总结与资源

Weaviate 是一个可扩展的、可用于生产的向量存储库。

这种集成使得 Weaviate 可以与 LangChain 一起使用，以增强大型语言模型的能力，并配备一个强大的数据存储库。其可扩展性和生产就绪性使其成为 LangChain 应用程序的向量存储库的绝佳选择，将缩短您的投产时间。