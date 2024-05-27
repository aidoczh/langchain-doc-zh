# Kinetica Vectorstore API

[Kinetica](https://www.kinetica.com/) 是一个集成了向量相似性搜索支持的数据库。

它支持以下功能：

- 精确和近似最近邻搜索

- L2 距离、内积和余弦距离

本文将展示如何使用 Kinetica 向量存储 (`Kinetica`)。

首先，需要一个 Kinetica 实例，可以按照这里的说明进行设置 - [安装说明](https://www.kinetica.com/developer-edition/)。

```python
# 安装所需的包
%pip install --upgrade --quiet langchain-openai
%pip install gpudb==7.2.0.1
%pip install --upgrade --quiet tiktoken
```
```output
[notice] A new release of pip is available: 23.2.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
注意：您可能需要重新启动内核以使用更新的包。
Requirement already satisfied: gpudb==7.2.0.0b in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (7.2.0.0b0)
Requirement already satisfied: future in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (from gpudb==7.2.0.0b) (0.18.3)
Requirement already satisfied: pyzmq in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (from gpudb==7.2.0.0b) (25.1.2)
[notice] A new release of pip is available: 23.2.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
注意：您可能需要重新启动内核以使用更新的包。
[notice] A new release of pip is available: 23.2.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
注意：您可能需要重新启动内核以使用更新的包。
```

我们想要使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```python
## 加载环境变量
from dotenv import load_dotenv
load_dotenv()
```
```output
False
```
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import (
    DistanceStrategy,
    Kinetica,
    KineticaSettings,
)
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
# Kinetica 需要与数据库建立连接。
# 这是如何设置连接的方法。
HOST = os.getenv("KINETICA_HOST", "http://127.0.0.1:9191")
USERNAME = os.getenv("KINETICA_USERNAME", "")
PASSWORD = os.getenv("KINETICA_PASSWORD", "")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")
def create_config() -> KineticaSettings:
    return KineticaSettings(host=HOST, username=USERNAME, password=PASSWORD)
```

## 使用欧几里得距离进行相似性搜索（默认）

```python
# Kinetica 模块将尝试创建一个与集合同名的表。
# 因此，请确保集合名称是唯一的，并且用户有创建表的权限。
COLLECTION_NAME = "state_of_the_union_test"
connection = create_config()
db = Kinetica.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    config=connection,
)
```
```python
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db.similarity_search_with_score(query)
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
Score:  0.6077010035514832
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6077010035514832
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6596046090126038
A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. 
And if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. 
We can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  
We’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  
We’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. 
We’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6597143411636353
A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. 
And if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. 
We can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  
We’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  
We’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. 
We’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.
--------------------------------------------------------------------------------
## 最大边际相关性搜索（MMR）
最大边际相关性搜索旨在优化与查询的相似性和所选文档之间的多样性。
```python
docs_with_score = db.max_marginal_relevance_search_with_score(query)
```
```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("得分：", score)
    print(doc.page_content)
    print("-" * 80)
```
```output

--------------------------------------------------------------------------------

得分： 0.6077010035514832

今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。还有，通过《披露法案》，这样美国人就可以知道是谁资助了我们的选举。

今晚，我想向一个致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名人选担任美国最高法院的法官。

而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分： 0.6852865219116211

这将改变美国，让我们走上赢得21世纪经济竞争的道路，尤其是与世界其他地方，特别是中国的竞争。

正如我告诉习近平的，押美国人民永远不是一个明智的赌注。

我们将为数百万美国人创造就业机会，现代化美国各地的道路、机场、港口和水路。

我们将做到这一切，以抵御气候危机的破坏影响，并促进环境正义。

我们将建立一个全国范围的50万个电动汽车充电站网络，开始更换有毒的铅水管，这样每个孩子和每个美国人在家和学校都有干净的饮用水，为每个美国人提供负担得起的高速互联网——城市、郊区、农村和部落社区。

已经宣布了4000个项目。

今晚，我宣布，今年我们将开始修复超过65,000英里的公路和1,500座失修的桥梁。

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分： 0.6866700053215027

我们无法改变我们的分歧有多深。但我们可以改变我们如何共同前进——在应对COVID-19和其他必须共同面对的问题上。

我最近访问了纽约市警察局，那是在威尔伯特·莫拉警官和他的搭档杰森·里维拉警官的葬礼后的几天。

他们在应对一起911报警时，被一名持有偷来的枪支的男子射杀。

莫拉警官27岁。

里维拉警官22岁。

他们都是多米尼加裔美国人，在同一条街道长大，后来选择作为警察巡逻。

我和他们的家人交谈，并告诉他们，我们永远感激他们的牺牲，我们将继续履行他们恢复每个社区应得的信任和安全的使命。

我长期致力于这些问题。

我知道什么是有效的：投资于犯罪预防和社区警务人员，他们将巡逻，了解社区，恢复信任和安全。

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

得分： 0.6936529278755188

但长期暴露于燃烧坑导致希斯的肺部和身体遭受了癌症。

丹妮尔说希斯一直是一位斗士，直到最后。

他不知道如何停止战斗，她也是如此。

在她的痛苦中，她找到了要求我们做得更好的目标。

今晚，丹妮尔——我们正在做到。

退伍军人事务部正在开创将有毒暴露与疾病联系起来的新方法，已经帮助更多的退伍军人获得福利。

今晚，我宣布，我们将扩大资格，使患有九种呼吸道癌症的退伍军人有资格获得福利。

我还呼吁国会：通过一项法律，确保在伊拉克和阿富汗受到有毒暴露影响的退伍军人最终获得他们应得的福利和全面医疗保健。

第四，让我们结束目前的癌症。

这对我和吉尔、卡玛拉以及你们许多人来说都是私人的。

癌症是美国的第二大死因——仅次于心脏病。

```
## 使用 vectorstore 进行工作
上面，我们从头开始创建了一个 vectorstore。然而，通常我们希望使用现有的 vectorstore。
为了做到这一点，我们可以直接初始化它。
```python
store = Kinetica(
    collection_name=COLLECTION_NAME,
    config=connection,
    embedding_function=embeddings,
)
```
### 添加文档
我们可以将文档添加到现有的向量存储库中。
```python
store.add_documents([Document(page_content="foo")])
```
```output

['b94dc67c-ce7e-11ee-b8cb-b940b0e45762']

```
```python
docs_with_score = db.similarity_search_with_score("foo")
```
```python
docs_with_score[0]
```
```output

(Document(page_content='foo'), 0.0)

```
```python
docs_with_score[1]
```
```output

(Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': '../../how_to/state_of_the_union.txt'}),

 0.6946534514427185)

```
### 覆盖向量存储库
如果您有现有的集合，可以通过执行 `from_documents` 并设置 `pre_delete_collection` = True 来覆盖它。
```python
db = Kinetica.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    config=connection,
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

(Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': '../../how_to/state_of_the_union.txt'}),

 0.6946534514427185)

```
### 使用向量存储库作为检索器
```python
retriever = store.as_retriever()
```
```python
print(retriever)
```
```output

tags=['Kinetica', 'OpenAIEmbeddings'] vectorstore=<langchain_community.vectorstores.kinetica.Kinetica object at 0x7f1644375e20>

```