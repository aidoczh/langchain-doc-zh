# 灯笼

[Lantern](https://github.com/lanterndata/lantern) 是一个用于 `Postgres` 的开源向量相似度搜索工具。

它支持：

- 精确和近似的最近邻搜索

- L2 平方距离、汉明距离和余弦距离

本笔记展示了如何使用 Postgres 向量数据库（`Lantern`）。

请参阅[安装说明](https://github.com/lanterndata/lantern#-quick-install)。

我们想要使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

# 安装必要的包

```python
!pip install openai
!pip install psycopg2-binary
!pip install tiktoken
```
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output
OpenAI API Key: ········
```
```python
## 加载环境变量
from typing import List, Tuple
from dotenv import load_dotenv
load_dotenv()
```
```output
False
```
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Lantern
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```
```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```
```python
# 灯笼需要数据库连接字符串。
# 例如 postgresql://postgres:postgres@localhost:5432/postgres
CONNECTION_STRING = getpass.getpass("DB Connection String:")
# # 或者，您可以从环境变量创建它。
# import os
# CONNECTION_STRING = Lantern.connection_string_from_db_params(
#     driver=os.environ.get("LANTERN_DRIVER", "psycopg2"),
#     host=os.environ.get("LANTERN_HOST", "localhost"),
#     port=int(os.environ.get("LANTERN_PORT", "5432")),
#     database=os.environ.get("LANTERN_DATABASE", "postgres"),
#     user=os.environ.get("LANTERN_USER", "postgres"),
#     password=os.environ.get("LANTERN_PASSWORD", "postgres"),
# )
# 或者您可以通过 `LANTERN_CONNECTION_STRING` 环境变量传递它
```
```output
DB Connection String: ········
```

## 使用余弦距离进行相似度搜索（默认）

```python
# 灯笼模块将尝试创建一个以集合名称命名的表。
# 因此，请确保集合名称是唯一的，并且用户有权限创建表。
COLLECTION_NAME = "state_of_the_union_test"
db = Lantern.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    pre_delete_collection=True,
)
```
```python
query = "总统对 Ketanji Brown Jackson 有什么看法"
docs_with_score = db.similarity_search_with_score(query)
```
```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
得分:  0.18440479
今晚，我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。还有，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统的最重要的宪法责任之一是提名人选担任美国最高法院法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将延续布雷耶司法部长的卓越传统。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.21727282
曾在私人执业中担任顶级诉讼律师。曾担任联邦公共辩护人。来自一家公立学校教育工作者和警察家庭。一个建立共识的人。自提名以来，她得到了广泛的支持——从警察工会到由民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们可以做到。在我们的边境，我们安装了新技术，如先进的扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地得到审理。
我们正在承诺并支持南美和中美的伙伴，以接纳更多的难民并保护他们的边界。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.22621095
对于我们的 LGBTQ+ 美国人，让我们最终将两党支持的《平等法案》送到我的办公桌上。针对跨性别美国人及其家人的一系列州法律的袭击是错误的。
正如我去年所说的，尤其是对我们年轻的跨性别美国人，作为你们的总统，我将永远支持你们，这样你们就可以做自己，实现上帝赋予你们的潜力。
虽然我们经常看起来似乎永远无法达成一致，但这并不是真的。去年，我签署了80项两党法案。从防止政府关门到保护亚裔免受仍然普遍存在的仇恨犯罪，再到改革军事司法。
很快，我们将加强我三十年前首次起草的《反对妇女暴力法案》。对我们来说，向国家展示我们可以团结一致，做出重大成就是很重要的。
因此，今晚我提出了一个国家的团结议程。我们可以一起做四件大事。
首先，战胜阿片类药物流行病。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
得分:  0.22654456
今晚，我宣布要打击这些公司对美国企业和消费者的过度收费。
随着华尔街公司接管更多的养老院，这些养老院的质量下降了，成本却上升了。
这种情况在我任内将结束。
医保将为养老院设定更高的标准，并确保您所爱的人得到他们应得和期望的照顾。
我们还将通过给予工人公平机会、提供更多培训和学徒制度，根据他们的技能而不是学位来雇佣他们，来降低成本并保持经济强劲发展。
让我们通过通过《工资公平法案》和带薪休假来提高最低工资至每小时15美元，并延长儿童税收抵免，这样没有人需要在贫困中抚养家庭。
让我们增加奖学金和增加我们对 HBCU 的历史支持，并投资于吉尔——我们的第一夫人，她全职教书——所称之为美国最好的秘密：社区学院。
--------------------------------------------------------------------------------
## 最大边际相关性搜索（MMR）
最大边际相关性搜索旨在优化与查询的相似性和所选文档之间的多样性。
```python
docs_with_score = db.max_marginal_relevance_search_with_score(query)
```
```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output

--------------------------------------------------------------------------------

得分：0.18440479

今晚。我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且，在此期间，通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。

今晚，我想向一个致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。

总统担负着最严肃的宪法责任之一，那就是提名某人担任美国最高法院的法官。

而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。 

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分：0.23515457

我们无法改变我们的分歧。但我们可以改变我们如何共同前进——在应对COVID-19和其他必须共同面对的问题上。

我最近访问了纽约市警察局，这是在威尔伯特·莫拉警官和他的搭档杰森·里维拉警官的葬礼后的几天。他们在响应一起911报警时，一名男子用一把偷来的枪射杀了他们。

莫拉警官27岁。

里维拉警官22岁。

他们都是多米尼加裔美国人，在同一条街道上长大，后来选择作为警察巡逻他们曾经生活的街道。

我与他们的家人交谈，并告诉他们，我们永远对他们的牺牲感激不尽，我们将继续履行他们恢复每个社区应有的信任和安全的使命。

我长期致力于这些问题。

我知道什么是有效的：投资于犯罪预防和社区警务人员，他们将巡逻，了解社区，并能恢复信任和安全。

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分：0.24478757

有人驻扎在基地，呼吸着焚烧战争废物的有毒烟雾——包括医疗和危险物质、喷气燃料等。

当他们回家时，许多世界上最健壮、训练最好的战士再也不是原来的自己。

头痛。麻木。头晕。

一种癌症会让他们躺在用国旗覆盖的棺材中。

我知道。

我的儿子贝奥中校就是其中之一。

我们不确定焚烧坑是他患脑癌的原因，或者是我们的许多军人患病的原因。

但我承诺要尽一切努力找出我们能找到的一切信息。

致力于像俄亥俄州的丹妮尔·罗宾逊这样的军人家庭。

她是希斯·罗宾逊一等军士的遗孀。

他生来就是一名士兵。陆军国民警卫队。科索沃和伊拉克的战地医生。

驻扎在巴格达附近，距离足球场大小的焚烧坑只有几码。

希斯的遗孀丹妮尔今晚和我们在一起。他们喜欢去俄亥俄州立大学的橄榄球比赛。他喜欢和女儿一起搭积木。

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分：0.25137997

我正在采取有力行动，确保我们的制裁对俄罗斯经济造成的痛苦是有针对性的。我将利用我们掌握的一切工具来保护美国企业和消费者。

今晚，我可以宣布，美国已与其他30个国家合作，从世界各地的储备中释放了6000万桶石油。

美国将领导这一努力，从我们自己的战略石油储备中释放3000万桶。如果有必要，我们将与我们的盟友团结一致，准备采取更多行动。

这些举措将有助于减轻国内的汽油价格。我知道关于正在发生的事情的消息似乎令人担忧。

但我想让你知道，我们会没事的。

当这个时代的历史被书写时，普京对乌克兰的战争将使俄罗斯变得更加脆弱，而世界其他地方将变得更加强大。

尽管人们不应该等到发生如此可怕的事情才能看清现在的利害关系，但现在每个人都清楚地看到了。

```
## 与 vectorstore 协作
上文中，我们从头开始创建了一个 vectorstore。然而，通常情况下我们希望使用现有的 vectorstore。
为了做到这一点，我们可以直接初始化它。
```python
store = Lantern(
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    embedding_function=embeddings,
)
```
### 添加文档
我们可以将文档添加到现有的向量存储中。
```python
store.add_documents([Document(page_content="foo")])
```
```output

['f8164598-aa28-11ee-a037-acde48001122']

```
```python
docs_with_score = db.similarity_search_with_score("foo")
```
```python
docs_with_score[0]
```
```output

(Document(page_content='foo'), -1.1920929e-07)

```
```python
docs_with_score[1]
```
```output

(Document(page_content='当大多数工人想要组建工会时，让我们通过PRO法案，他们不应该被阻止。\n\n当我们投资于我们的工人，当我们共同从底层和中间层建设经济时，我们可以做到长期以来未曾做到的事情：建设一个更美好的美国。\n\n在过去两年多的时间里，COVID-19影响了我们生活中的每一个决策和整个国家的生活。\n\n我知道你们感到疲倦、沮丧和筋疲力尽。\n\n但我也知道这一点。\n\n由于我们取得的进展，由于你们的韧性和我们拥有的工具，今晚我可以说我们正在安全地向前迈进，回到更正常的生活节奏。\n\n我们已经达到了抗击COVID-19的新时刻，严重病例下降到去年7月以来未见的水平。\n\n就在几天前，疾病控制与预防中心（CDC）发布了新的口罩指南。\n\n根据这些新指南，在大多数国家的大多数美国人现在可以不戴口罩。', metadata={'source': '../../how_to/state_of_the_union.txt'}),

 0.24038416)

```
### 覆盖向量存储
如果您有一个现有的集合，您可以通过执行`from_documents`并设置`pre_delete_collection=True`来覆盖它
这将在重新填充之前删除集合
```python
db = Lantern.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    connection_string=CONNECTION_STRING,
    pre_delete_collection=True,
)
```
```python
docs_with_score = db.similarity_search_with_score("foo")
```
```python
docs_with_score[0]
```
```output

(Document(page_content='当大多数工人想要组建工会时，让我们通过PRO法案，他们不应该被阻止。\n\n当我们投资于我们的工人，当我们共同从底层和中间层建设经济时，我们可以做到长期以来未曾做到的事情：建设一个更美好的美国。\n\n在过去两年多的时间里，COVID-19影响了我们生活中的每一个决策和整个国家的生活。\n\n我知道你们感到疲倦、沮丧和筋疲力尽。\n\n但我也知道这一点。\n\n由于我们取得的进展，由于你们的韧性和我们拥有的工具，今晚我可以说我们正在安全地向前迈进，回到更正常的生活节奏。\n\n我们已经达到了抗击COVID-19的新时刻，严重病例下降到去年7月以来未见的水平。\n\n就在几天前，疾病控制与预防中心（CDC）发布了新的口罩指南。\n\n根据这些新指南，在大多数国家的大多数美国人现在可以不戴口罩。', metadata={'source': '../../how_to/state_of_the_union.txt'}),

 0.2403456)

```
### 使用向量存储作为检索器
```python
retriever = store.as_retriever()
```
```python
print(retriever)
```
```output

tags=['Lantern', 'OpenAIEmbeddings'] vectorstore=<langchain_community.vectorstores.lantern.Lantern object at 0x11d02f9d0>

```