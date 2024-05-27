# 松果

[Pinecone](https://docs.pinecone.io/docs/overview) 是一个功能广泛的向量数据库。

这篇笔记展示了如何使用与 `Pinecone` 向量数据库相关的功能。

设置以下环境变量以便在本文档中进行操作：

- `OPENAI_API_KEY`：您的 OpenAI API 密钥，用于使用 `OpenAIEmbeddings`

```python
%pip install --upgrade --quiet  \
    langchain-pinecone \
    langchain-openai \
    langchain \
    pinecone-notebooks
```

迁移说明：如果您正在从 Pinecone 的 `langchain_community.vectorstores` 实现进行迁移，您可能需要在安装 `langchain-pinecone` 之前删除 `pinecone-client` v2 依赖，因为 `langchain-pinecone` 依赖于 `pinecone-client` v3。

首先，让我们将我们的国情咨文拆分成分块的 `docs`。

```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

现在让我们创建一个新的 Pinecone 账户，或者登录您现有的账户，并创建一个 API 密钥以在本笔记本中使用。

```python
from pinecone_notebooks.colab import Authenticate
Authenticate()
```

新创建的 API 密钥已存储在 `PINECONE_API_KEY` 环境变量中。我们将使用它来设置 Pinecone 客户端。

```python
import os
pinecone_api_key = os.environ.get("PINECONE_API_KEY")
pinecone_api_key
import time
from pinecone import Pinecone, ServerlessSpec
pc = Pinecone(api_key=pinecone_api_key)
```

接下来，让我们连接到您的 Pinecone 索引。如果名为 `index_name` 的索引不存在，将会被创建。

```python
import time
index_name = "langchain-index"  # 如果需要，可以更改
existing_indexes = [index_info["name"] for index_info in pc.list_indexes()]
if index_name not in existing_indexes:
    pc.create_index(
        name=index_name,
        dimension=1536,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
    )
    while not pc.describe_index(index_name).status["ready"]:
        time.sleep(1)
index = pc.Index(index_name)
```

现在我们的 Pinecone 索引已经设置好，我们可以使用 `PineconeVectorStore.from_documents` 将这些分块的文档作为内容进行更新。

```python
from langchain_pinecone import PineconeVectorStore
docsearch = PineconeVectorStore.from_documents(docs, embeddings, index_name=index_name)
```
```python
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs = docsearch.similarity_search(query)
print(docs[0].page_content)
```
```output
今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家最顶尖的法律智慧之一，将延续布雷耶司法部长的卓越传统。
```

### 向现有索引添加更多文本

可以使用 `add_texts` 函数将更多文本嵌入并更新到现有的 Pinecone 索引中。

```python
vectorstore = PineconeVectorStore(index_name=index_name, embedding=embeddings)
vectorstore.add_texts(["更多文本！"])
```
```output
['24631802-4bad-44a7-a4ba-fd71f00cc160']
```

### 最大边际相关搜索

除了在 retriever 对象中使用相似性搜索之外，您还可以使用 `mmr` 作为检索器。

```python
retriever = docsearch.as_retriever(search_type="mmr")
matched_docs = retriever.invoke(query)
for i, d in enumerate(matched_docs):
    print(f"\n## 文档 {i}\n")
    print(d.page_content)
```
```output
## 文档 0
今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家最顶尖的法律智慧之一，将延续布雷耶司法部长的卓越传统。
## 文档 1
我正在采取有力措施，确保我们的制裁痛击俄罗斯经济。我将利用我们掌握的一切工具来保护美国企业和消费者。
今晚，我可以宣布，美国已与其他30个国家合作，从世界各地的储备中释放了6000万桶石油。
美国将领导这一努力，从我们自己的战略石油储备中释放3000万桶。如果有必要，我们将与我们的盟友团结一致，做更多的事情。
这些举措将有助于减轻国内的汽油价格。我知道关于正在发生的事情的消息似乎令人担忧。
但我想让你知道，我们会没事的。
当这个时代的历史被书写时，普京对乌克兰的战争将使俄罗斯变得更加脆弱，而世界其他地方将变得更加强大。
尽管不应该需要发生如此可怕的事情，人们才能清楚地看到现在的利害关系。
## 文档 2
我们无法改变我们之间的分歧。但我们可以改变我们如何共同前进——在 COVID-19 和其他我们必须共同面对的问题上。
不久前，我参观了纽约市警察局，在警官威尔伯特·莫拉和他的搭档杰森·里维拉的葬礼后的几天。
他们在响应一个 911 呼叫时，被一个人用一把偷来的枪射杀。
莫拉警官 27 岁。
里维拉警官 22 岁。
两位多米尼加裔美国人，他们在同一条街道长大，后来选择作为警察巡逻。
我与他们的家人交谈，并告诉他们，我们永远对他们的牺牲感激，并且我们将继续履行他们恢复每个社区应得的信任和安全的使命。
我在这些问题上工作了很长时间。
我知道什么是有效的：投资于犯罪预防和社区警察，他们将巡逻，了解社区，并恢复信任和安全。
## 文档 3
有人驻扎在基地，呼吸着“燃烧坑”散发的有毒烟雾——焚烧战争废物、医疗和危险材料、喷气燃料等。
当他们回家时，许多世界上最健壮、训练有素的战士再也不是以前的自己。
头痛。麻木。头晕。
一种癌症会让他们躺在用国旗覆盖的棺材中。
我知道。
其中一名士兵是我的儿子博伊中校。
我们不确定燃烧坑是否导致了他的脑癌，或者导致了我们的许多军人的疾病。
但我承诺，我们将尽一切努力找出我们能找到的一切。
致力于像俄亥俄州的丹尼尔·罗宾逊这样的军人家庭。
他是希思·罗宾逊一等兵的遗孀。
他生来就是一名士兵。国民警卫队。科索沃和伊拉克的战地医生。
驻扎在距离足球场大小的燃烧坑几码远的巴格达附近。
希思的丈夫丹尼尔今晚和我们在一起。他们喜欢去俄亥俄州立大学的橄榄球比赛。他喜欢和他们的女儿一起搭积木。
```
```python
或者直接使用 `max_marginal_relevance_search` 函数：
```python

found_docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10)

for i, doc in enumerate(found_docs):

    print(f"{i + 1}.", doc.page_content, "\n")

```
```output

1. 今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。

2. 我们无法改变我们的分歧有多深。但我们可以改变我们如何共同面对COVID-19和其他必须共同应对的问题。

我最近访问了纽约市警察局，这是在威尔伯特·莫拉警官和他的搭档杰森·里维拉警官的葬礼后的几天。他们在响应一个报警电话时，一名男子用一把偷来的枪射杀了他们。

莫拉警官27岁。

里维拉警官22岁。

他们都是多米尼加裔美国人，在同一条街道长大，后来选择作为警察巡逻。我与他们的家人交谈，并告诉他们，我们永远感激他们的牺牲，我们将继续履行他们恢复每个社区应得的信任和安全的使命。

我长期致力于这些问题。

我知道什么是有效的：投资于犯罪预防和社区警务人员，他们将巡逻，了解社区，恢复信任和安全。

```