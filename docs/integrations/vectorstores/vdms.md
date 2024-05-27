# Intel 的视觉数据管理系统（VDMS）

> Intel 的 [VDMS](https://github.com/IntelLabs/vdms) 是一种用于高效访问大规模“视觉”数据的存储解决方案，旨在通过搜索以图形存储的视觉元数据来实现云规模，并实现对视觉数据的机器友好增强，以加快访问速度。VDMS 根据 MIT 许可证发布。

VDMS 支持：

- K 近邻搜索

- 欧氏距离（L2）和内积（IP）

- 用于索引和计算距离的库：TileDBDense、TileDBSparse、FaissFlat（默认）、FaissIVFFlat

- 向量和元数据搜索

VDMS 具有服务器和客户端组件。要设置服务器，请参阅[安装说明](https://github.com/IntelLabs/vdms/blob/master/INSTALL.md)或使用[docker 镜像](https://hub.docker.com/r/intellabs/vdms)。

本笔记本展示了如何使用 docker 镜像将 VDMS 用作向量存储。

要开始，请安装用于 VDMS 客户端和 Sentence Transformers 的 Python 包：

```python
# 安装必要的包
%pip install --upgrade --quiet pip sentence-transformers vdms "unstructured-inference==0.6.6";
```
```output
注意：您可能需要重新启动内核以使用更新后的包。
```

## 启动 VDMS 服务器

在这里，我们使用端口 55555 启动 VDMS 服务器。

```python
!docker run --rm -d -p 55555:55555 --name vdms_vs_test_nb intellabs/vdms:latest
```
```output
e6061b270eef87de5319a6c5af709b36badcad8118069a8f6b577d2e01ad5e2d
```

## 基本示例（使用 Docker 容器）

在这个基本示例中，我们演示了如何将文档添加到 VDMS 中并将其用作向量数据库。

您可以单独在 Docker 容器中运行 VDMS 服务器，以便与通过 VDMS Python 客户端连接到服务器的 LangChain 一起使用。

VDMS 有处理多个文档集的能力，但 LangChain 接口期望一个文档集，因此我们需要指定文档集的名称。LangChain 使用的默认文档集名称是“langchain”。

```python
import time
from langchain_community.document_loaders.text import TextLoader
from langchain_community.vectorstores import VDMS
from langchain_community.vectorstores.vdms import VDMS_Client
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters.character import CharacterTextSplitter
time.sleep(2)
DELIMITER = "-" * 50
# 连接到 VDMS 向量存储
vdms_client = VDMS_Client(host="localhost", port=55555)
```

以下是一些用于打印结果的辅助函数。

```python
def print_document_details(doc):
    print(f"内容：\n\t{doc.page_content}\n")
    print("元数据：")
    for key, value in doc.metadata.items():
        if value != "Missing property":
            print(f"\t{key}:\t{value}")
def print_results(similarity_results, score=True):
    print(f"{DELIMITER}\n")
    if score:
        for doc, score in similarity_results:
            print(f"得分：\t{score}\n")
            print_document_details(doc)
            print(f"{DELIMITER}\n")
    else:
        for doc in similarity_results:
            print_document_details(doc)
            print(f"{DELIMITER}\n")
def print_response(list_of_entities):
    for ent in list_of_entities:
        for key, value in ent.items():
            if value != "Missing property":
                print(f"\n{key}:\n\t{value}")
        print(f"{DELIMITER}\n")
```

### 加载文档并获取嵌入函数

在这里，我们加载最近的国情咨文并将文档拆分为块。

LangChain 向量存储使用字符串/关键字 `id` 用于文档管理。默认情况下，`id` 是一个 uuid，但在这里我们将其定义为一个整数转换为字符串。还为文档提供了额外的元数据，并且在此示例中使用 HuggingFaceEmbeddings 作为嵌入函数。

```python
# 加载文档并将其拆分为块
document_path = "../../how_to/state_of_the_union.txt"
raw_documents = TextLoader(document_path).load()
# 将其拆分为块
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(raw_documents)
ids = []
for doc_idx, doc in enumerate(docs):
    ids.append(str(doc_idx + 1))
    docs[doc_idx].metadata["id"] = str(doc_idx + 1)
    docs[doc_idx].metadata["page_number"] = int(doc_idx + 1)
    docs[doc_idx].metadata["president_included"] = (
        "president" in doc.page_content.lower()
    )
print(f"# 文档数：{len(docs)}")
# 创建开源嵌入函数
embedding = HuggingFaceEmbeddings()
print(
    f"# 嵌入维度：{len(embedding.embed_query('This is a test document.'))}"
)
```
```output
# 文档数：42
# 嵌入维度：768
```

### 使用 Faiss Flat 和欧氏距离（默认）进行相似性搜索

在这一部分，我们使用 FAISS IndexFlat 索引（默认）和欧几里得距离（默认）作为相似性搜索的距离度量，将文档添加到 VDMS 中。我们搜索与查询“总统对Ketanji Brown Jackson说了什么”相关的三个文档（`k=3`）。

```python
# 添加数据
collection_name = "my_collection_faiss_L2"
db = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name=collection_name,
    embedding=embedding,
)
# 查询（无元数据过滤）
k = 3
query = "What did the president say about Ketanji Brown Jackson"
returned_docs = db.similarity_search(query, k=k, filter=None)
print_results(returned_docs, score=False)
```
```output
--------------------------------------------------
内容：
    今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
    今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
    总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
    而我在4天前就做到了，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续布雷耶司法部长的卓越传统。
元数据：
    id:	32
    page_number:	32
    president_included:	True
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
内容：
    正如弗朗西斯·豪根今晚所展示的，我们必须让社交媒体平台为他们为了利润而对我们的孩子进行的全国性实验负责。
    是时候加强隐私保护，禁止针对儿童的定向广告，要求科技公司停止收集我们孩子的个人数据。
    让我们为所有美国人提供他们所需的心理健康服务。让更多人可以寻求帮助，并实现身心健康的全面平等。
    第三，支持我们的退伍军人。
    退伍军人是我们中最优秀的人。
    我一直相信我们有一个神圣的责任，那就是装备我们派往战场的所有人，并在他们回家后照顾他们和他们的家人。
    我的政府正在提供工作培训和住房援助，并且现在帮助低收入退伍军人无债地获得退伍军人事务部的医疗照顾。
    我们在伊拉克和阿富汗的部队面临着许多危险。
元数据：
    id:	37
    page_number:	37
    president_included:	False
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
内容：
    以前曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个共识的建设者。自她被提名以来，她得到了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。
    而如果我们要推进自由和正义，我们需要保护边境并修复移民系统。
    我们可以做到两者。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测走私毒品。
    我们与墨西哥和危地马拉进行了联合巡逻，以抓捕更多的人口贩子。
    我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。
    我们正在承诺并支持南美和中美的伙伴，以接纳更多的难民并保护他们自己的边界。
元数据：
    id:	33
    page_number:	33
    president_included:	False
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
```
```python
# 查询（带过滤）
k = 3
constraints = {"page_number": [">", 30], "president_included": ["==", True]}
query = "What did the president say about Ketanji Brown Jackson"
returned_docs = db.similarity_search(query, k=k, filter=constraints)
print_results(returned_docs, score=False)
```
```output
--------------------------------------------------
内容：
    今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯投票权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
    今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是即将退休的美国最高法院大法官。布雷耶司法部长，感谢您的服务。
    总统的最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
    而我在4天前就做到了，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智慧之一，将继续布雷耶司法部长的卓越传统。
元数据：
    id:	32
    page_number:	32
    president_included:	True
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
内容：
    而对于我们的 LGBTQ+ 美国人，让我们最终将两党的《平等法案》送到我的办公桌上。那些针对跨性别美国人及其家庭的州法律的猛烈攻击是错误的。
    正如我去年所说，特别是对我们年轻的跨性别美国人，作为你们的总统，我将永远支持你们，这样你们可以做自己，并发挥你们上帝赋予的潜力。
    尽管我们经常看起来似乎永远不会达成一致，但这并不是真的。去年，我签署了80项两党法案。从防止政府关门到保护亚裔免受仍然太常见的仇恨犯罪，再到改革军事司法。
    而且很快，我们将加强我三十年前首次起草的《反对妇女暴力法》。对我们来说，向国家展示我们可以团结起来做大事情是很重要的。
元数据：
    id:	35
    page_number:	35
    president_included:	True
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
内容：
    上个月，我宣布了我们计划加速实施奥巴马总统六年前要我领导的“癌症登月计划”。
    我们的目标是在未来25年内将癌症死亡率至少降低50%，将更多的癌症从死刑变为可治疗的疾病。
    为患者和家庭提供更多支持。
    为了实现这一目标，我呼吁国会资助ARPA-H，即卫生高级研究项目署。它基于DARPA——国防部的项目，该项目导致了互联网、GPS等许多成果。
    ARPA-H将有一个独特的目标——推动癌症、老年痴呆症、糖尿病等方面的突破。
    一个国家的团结议程。
    我们可以做到。
    我的美国同胞们——今晚，我们聚集在一个神圣的空间——我们民主的堡垒。在这个国会大厦里，一代又一代的美国人在巨大的争议中讨论重大问题，并做出了伟大的事情。
    我们为自由而战，扩大自由，打败极权主义和恐怖主义。
元数据：
    id:	40
    page_number:	40
    president_included:	True
    source:	../../how_to/state_of_the_union.txt
--------------------------------------------------
```

### 使用 TileDBDense 和欧氏距离进行相似性搜索

在本节中，我们使用 TileDB Dense 索引和 L2 作为相似性搜索的距离度量，将文档添加到 VDMS 中。我们搜索与查询“总统对 Ketanji Brown Jackson 说了什么”相关的三个文档（k=3），并返回得分和文档。

```python
db_tiledbD = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name="my_collection_tiledbD_L2",
    embedding=embedding,
    engine="TileDBDense",
    distance_strategy="L2",
)
k = 3
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db_tiledbD.similarity_search_with_score(query, k=k, filter=None)
print_results(docs_with_score)
```
```output
--------------------------------------------------
得分：1.2032090425491333
内容：
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。顺便说一下，通过《披露法案》，这样美国人就可以知道谁资助了我们的选举。
今晚，我想向一个致力于为这个国家服务的人表示敬意：司法部长斯蒂芬·布雷耶尔——一位陆军退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶尔法官，感谢您的服务。
总统的最重要的宪法责任之一是提名人选担任美国最高法院法官。
4天前，我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶级的法律智慧之一，将继续布雷耶尔法官的卓越传统。
元数据：
id: 32
页码: 32
包含总统: 是
来源: ../../how_to/state_of_the_union.txt
--------------------------------------------------
得分：1.495247483253479
内容：
正如弗朗西斯·豪根今晚与我们在一起所展示的，我们必须让社交媒体平台为他们在我们的孩子身上进行的以盈利为目的的全国性实验负责。
是时候加强隐私保护，禁止向儿童进行定向广告，要求科技公司停止收集我们孩子的个人数据。
让我们为所有美国人提供他们所需的心理健康服务。让他们有更多可以求助的人，并实现身心健康护理的全面平等。
第三，支持我们的退伍军人。
退伍军人是我们中最优秀的人。
我一直相信，我们有一个神圣的责任，即在我们派遣他们上战场时为他们提供装备，并在他们回家时照顾他们和他们的家人。
我的政府正在提供就业培训和住房援助，现在还帮助低收入退伍军人免除退伍军人事务部的债务。
我们在伊拉克和阿富汗的部队面临许多危险。
元数据：
id: 37
页码: 37
包含总统: 否
来源: ../../how_to/state_of_the_union.txt
--------------------------------------------------
得分：1.5008409023284912
内容：
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护人。来自公立学校教育工作者和警察家庭。是一个共识的建设者。自从她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民系统。
我们可以两者兼顾。在我们的边境，我们安装了新技术，如先进的扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。
我们正在与南美和中美洲的合作伙伴达成承诺，以接纳更多的难民并保护他们的边境。
元数据：
id: 33
页码: 33
包含总统: 否
来源: ../../how_to/state_of_the_union.txt
--------------------------------------------------
```

### 使用 Faiss IVFFlat 和欧氏距离进行相似性搜索

在本节中，我们使用 Faiss IndexIVFFlat 索引和 L2 作为相似性搜索的距离度量，将文档添加到 VDMS 中。我们搜索与查询“总统对 Ketanji Brown Jackson 说了什么”相关的三个文档（k=3），并返回得分和文档。

```python
db_FaissIVFFlat = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name="my_collection_FaissIVFFlat_L2",
    embedding=embedding,
    engine="FaissIVFFlat",
    distance_strategy="L2",
)
# 查询
k = 3
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db_FaissIVFFlat.similarity_search_with_score(query, k=k, filter=None)
print_results(docs_with_score)
```
```output
```markdown

# 更新和删除

在构建真正的应用程序时，您希望不仅添加数据，还要更新和删除数据。

以下是一个基本示例，展示了如何执行这些操作。首先，我们将更新与查询最相关的文档的元数据。

```python
doc = db.similarity_search(query)[0]
print(f"原始元数据：\n\t{doc.metadata}")
# 更新文档的元数据
doc.metadata["new_value"] = "你好，世界"
print(f"新的元数据：\n\t{doc.metadata}")
print(f"{DELIMITER}\n")
# 在 VDMS 中更新文档
id_to_update = doc.metadata["id"]
db.update_document(collection_name, id_to_update, doc)
response, response_array = db.get(
    collection_name, constraints={"id": ["==", id_to_update]}
)
# 显示结果
print(f"更新后的条目（id={id_to_update}）：")
print_response([response[0]["FindDescriptor"]["entities"][0]])
```
```output
原始元数据： 
	{'id': '32', 'page_number': 32, 'president_included': True, 'source': '../../how_to/state_of_the_union.txt'}
新的元数据： 
	{'id': '32', 'page_number': 32, 'president_included': True, 'source': '../../how_to/state_of_the_union.txt', 'new_value': '你好，世界'}
--------------------------------------------------
更新后的条目（id=32）：
内容：
	今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
```

今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前提名了联邦上诉法院法官凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）。她是我国顶尖的法律专家之一，将延续布雷耶司法部长卓越的传统。

id: 32

new_value: hello world

page_number: 32

president_included: True

source: ../../how_to/state_of_the_union.txt

![图片](https://example.com/image.png)

VDMS支持各种类型的视觉数据和操作。一些功能已集成在LangChain接口中，但随着VDMS的不断发展，将添加更多工作流改进。

集成到LangChain中的其他功能如下。

### 通过向量进行相似性搜索

您可以通过嵌入/向量进行搜索，而不是通过字符串查询。

```python
embedding_vector = embedding.embed_query(query)
returned_docs = db.similarity_search_by_vector(embedding_vector)
# 打印结果
print_document_details(returned_docs[0])
```

Content:

今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个终身致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我国顶尖的法律专家之一，将延续布雷耶司法部长卓越的传统。

元数据:

id: 32

new_value: hello world

page_number: 32

president_included: True

source: ../../how_to/state_of_the_union.txt

今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《披露法案》，这样美国人就能知道谁在资助我们的选举。

今晚，我想向一个毕生致力于为国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶司法部长的卓越传统。

元数据：

id: 32

new_value: hello world

page_number: 32

president_included: True

source: ../../how_to/state_of_the_union.txt

#### 最大边际相关性搜索（MMR）

除了在检索器对象中使用相似性搜索外，您还可以使用 `mmr`。

```python
retriever = db.as_retriever(search_type="mmr")
relevant_docs = retriever.invoke(query)[0]
print_document_details(relevant_docs)
```
```output
内容：
今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《披露法案》，这样美国人就能知道谁在资助我们的选举。
今晚，我想向一个毕生致力于为国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶司法部长的卓越传统。
元数据：
id: 32
new_value: hello world
page_number: 32
president_included: True
source: ../../how_to/state_of_the_union.txt
```

我们也可以直接使用 MMR。

```python
mmr_resp = db.max_marginal_relevance_search_with_score(query, k=2, fetch_k=10)
print_results(mmr_resp)
```
```output
--------------------------------------------------
得分：1.2032092809677124
内容：
今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。顺便一提，通过《披露法案》，这样美国人就能知道谁在资助我们的选举。
今晚，我想向一个毕生致力于为国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智囊，将继续延续布雷耶司法部长的卓越传统。
元数据：
id: 32
new_value: hello world
page_number: 32
president_included: True
source: ../../how_to/state_of_the_union.txt
--------------------------------------------------
得分：1.507053256034851
内容：
但长期接触燃烧坑产生的癌症摧毁了希思的肺部和身体。
丹妮尔说希思一直是个斗士。
他不知道如何停止战斗，她也一样。
在痛苦中，她找到了要求我们做得更好的目的。
今晚，丹妮尔——我们正在做。
退伍军人事务部正在开创将有毒暴露与疾病联系起来的新方法，已经帮助更多退伍军人获得福利。
今晚，我宣布我们将扩大对患有九种呼吸系统癌症的退伍军人的资格。
我还呼吁国会：通过一项法律，确保在伊拉克和阿富汗受有毒暴露影响的退伍军人最终获得他们应得的福利和全面医疗保健。
第四，让我们结束目前的癌症。
这对我和吉尔、卡玛拉以及你们许多人来说都是个人问题。
癌症是美国的第二大死因——仅次于心脏病。
元数据：
id: 39
page_number: 39
president_included: False
source: ../../how_to/state_of_the_union.txt
--------------------------------------------------
```

### 删除集合

之前，我们根据其 `id` 删除了文档。在这里，由于未提供 ID，所有文档都被删除了。

```python
print("删除前的文档数：", db.count(collection_name))
db.delete(collection_name=collection_name)
print("删除后的文档数：", db.count(collection_name))
```
```output
删除前的文档数： 40
删除后的文档数： 0
```

## 停止 VDMS 服务器

```python
```markdown

!docker kill vdms_vs_test_nb

```
```markdown

huggingface/tokenizers: 当前进程在并行处理已经被使用后进行了 fork。为避免死锁，禁用并行处理…

要禁用此警告，您可以：

	- 如果可能的话，在 fork 之前避免使用 `tokenizers`

	- 明确设置环境变量 TOKENIZERS_PARALLELISM=(true | false)

```
```markdown

vdms_vs_test_nb

```