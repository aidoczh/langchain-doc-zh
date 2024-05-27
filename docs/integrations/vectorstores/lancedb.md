# LanceDB

[LanceDB](https://lancedb.com/) 是一个基于持久存储构建的用于向量搜索的开源数据库，极大地简化了嵌入式的检索、过滤和管理。完全开源。

本笔记展示了如何使用与 `LanceDB` 向量数据库相关的功能，基于 Lance 数据格式。

```python
! pip install -U langchain-openai
```

```python
! pip install lancedb
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import LanceDB
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
documents = CharacterTextSplitter().split_documents(documents)
embeddings = OpenAIEmbeddings()
```

##### 对于 LanceDB 云端，可以按以下方式调用向量存储：

```python
db_url = "db://lang_test"  # 您创建的数据库的 URL
api_key = "xxxxx"  # 您的 API 密钥
region="us-east-1-dev"  # 您选择的地区
vector_store = LanceDB(
    uri=db_url,
    api_key=api_key,
    region=region,
    embedding=embeddings,
    table_name='langchain_test'
    )
```

```python
docsearch = LanceDB.from_documents(documents, embeddings)
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = docsearch.similarity_search(query)
```

此外，要探索表格，可以将其加载到数据框中或将其保存在 csv 文件中：

```python
tbl = docsearch.get_table()
print("tbl:", tbl)
pd_df = tbl.to_pandas()
# pd_df.to_csv("docsearch.csv", index=False)
# 您还可以使用旧的连接对象创建一个新的向量存储对象：
vector_store = LanceDB(connection=tbl, embedding=embeddings)
```

```python
print(docs[0].page_content)
```

```output
他们正在响应 9-1-1 的呼叫时，一个男子用一把偷来的枪杀死了他们。
Mora 警官 27 岁。
Rivera 警官 22 岁。
两位多米尼加裔美国人，在他们后来选择巡逻的街道上长大。
我与他们的家人交谈，并告诉他们我们永远欠他们的牺牲，我们将继续履行他们恢复每个社区应得的信任和安全的使命。
我已经在这些问题上工作了很长时间。
我知道什么是有效的：投资于犯罪预防和社区警务人员，他们将巡逻，了解社区，并能恢复信任和安全。
因此，让我们不要放弃我们的街道。或者在安全和平等正义之间做出选择。
让我们团结起来，保护我们的社区，恢复信任，并要求执法机构对其负责。
这就是为什么司法部要求其官员佩戴身体摄像头，禁止使用致命夹颈，并限制无敲门令。
这就是为什么美国拯救计划提供了 3500 亿美元，供城市、州和县使用，以雇佣更多警察，并投资于像社区暴力中断这样的经过验证的策略——值得信赖的信使打破暴力和创伤的循环，并给年轻人带来希望。
我们都应该同意：答案不是削减警察的经费。答案是为警察提供他们保护社区所需的资源和培训。
我要求民主党和共和党一样：通过我的预算，保护我们的社区。
我将竭尽全力打击枪支走私和您可以在线购买并在家制造的无法追踪的幽灵枪——它们没有序列号。
我要求国会通过已被证明可以减少枪支暴力的措施。通过普遍背景检查。为什么恐怖分子名单上的任何人都可以购买武器？
禁止突击武器和大容量弹匣。
废除让枪支制造商成为美国唯一不受起诉的行业的责任保护。
这些法律不侵犯第二修正案。它们挽救生命。
美国最基本的权利是投票权——并且让它得到计数。它正受到攻击。
在一个又一个州，不仅通过了新法律来压制选票，而且颠覆了整个选举。
我们不能让这种事发生。
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。而且在此期间，通过《披露法案》，这样美国人就能知道是谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人致敬：司法部大法官史蒂芬·布雷耶——一名陆军退伍军人、宪法学者，也是美国最高法院的退休大法官。司法部大法官布雷耶，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院的大法官。
而我在 4 天前就做到了，当我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续布雷耶大法官的卓越传统。
她曾是私人执业的顶级诉讼律师。曾是联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。一个建立共识的人。自从她被提名以来，她得到了广泛的支持——从警察工会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。
```

```python
print(docs[0].page_content)
```

```output
他们正在应对一起 9-1-1 报警时，一名男子用一把偷来的枪射杀了他们。
莫拉警官 27 岁。
里维拉警官 22 岁。
两人都是多米尼加裔美国人，在同一条街道长大，后来选择成为警察。
我与他们的家人交谈，并告诉他们，我们永远感激他们的牺牲，我们将继续履行他们的使命，恢复每个社区应有的信任和安全。
我长期致力于这些问题。
我知道什么是有效的：投资于犯罪预防和社区警务人员，他们会巡逻，了解社区，恢复信任和安全。
所以，让我们不要放弃我们的街道。也不要在安全和平等正义之间做选择。
让我们团结起来，保护我们的社区，恢复信任，并要求执法机构承担责任。
这就是为什么司法部要求其警官配备身体摄像头，禁止使用致命夹颈手法，并限制无敲门搜查令。
这就是为什么《美国拯救计划》提供了 3500 亿美元，供城市、州和县使用，以雇佣更多警察，并投资于像社区暴力干预这样经过验证的策略——信任的传递者打破暴力和创伤的循环，给年轻人带来希望。
我们都应该认同：答案不是削减警察经费。答案是为警察提供他们保护社区所需的资源和培训。
我请求民主党和共和党一样：通过我的预算案，保障我们的社区安全。
我将竭尽全力打击枪支走私和网购制造的无法追踪的散弹枪。
我请求国会通过已被证明有效的措施来减少枪支暴力。通过普遍背景调查。为什么恐怖分子名单上的任何人都能购买武器呢？
禁止突击武器和大容量弹匣。
废除让枪支制造商成为美国唯一不受起诉的行业的责任保护。
这些法律不侵犯第二修正案。它们挽救生命。
美国最基本的权利是投票权，以及让它得到统计。而这些权利正在受到威胁。
在一个又一个州，不仅通过了新法律来压制选举权，而且颠覆了整个选举。
我们不能让这种事发生。
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。而且，顺便再通过《揭露法案》，这样美国人就能知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部大法官斯蒂芬·布雷耶——一名陆军退伍军人，宪法学者，美国最高法院即将退休的大法官。斯蒂芬·布雷耶大法官，感谢您的服务。
总统最严肃的宪法责任之一是提名某人担任美国最高法院的大法官。
而我在 4 天前就做到了，当我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续斯蒂芬·布雷耶大法官的卓越传统。
她曾是一名顶尖的私人律师。曾是一名联邦公共辩护人。来自一家公立学校教育工作者和警察的家庭。她是一名建立共识的人。自从她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要保护边境并修复移民制度。
我们可以做到。在我们的边境，我们安装了新技术，如尖端扫描仪，以更好地检测走私毒品。
我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。
我们正在设立专门的移民法官，以便那些逃离迫害和暴力的家庭能够更快地得到审理。
```

```python
print(docs[0].metadata)
```