# LLMLingua文件压缩器

[LLMLingua](https://github.com/microsoft/LLMLingua)利用紧凑、经过良好训练的语言模型（例如GPT2-small，LLaMA-7B）来识别并删除提示中的非必要标记。这种方法使得大型语言模型（LLMs）的推理变得高效，最多可实现20倍的压缩，且性能损失最小。

这份笔记展示了如何使用LLMLingua作为文档压缩器。

```python
%pip install --upgrade --quiet  llmlingua accelerate
```

```output
[notice] 可用新版本的pip：23.3.2 -> 24.0
[notice] 要更新，请运行：python -m pip install --upgrade pip
注意：您可能需要重新启动内核以使用更新后的软件包。
```

```python
# 用于打印文档的辅助函数
def pretty_print_docs(docs):
    print(
        f"\n{'-' * 100}\n".join(
            [f"Document {i+1}:\n\n" + d.page_content for i, d in enumerate(docs)]
        )
    )
```

## 设置基本向量存储检索器

让我们从初始化一个简单的向量存储检索器开始，并存储2023年国情咨文（分块）。我们可以设置检索器以检索大量（20个）文档。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
documents = TextLoader(
    "../../how_to/state_of_the_union.txt",
).load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
texts = text_splitter.split_documents(documents)
embedding = OpenAIEmbeddings(model="text-embedding-ada-002")
retriever = FAISS.from_documents(texts, embedding).as_retriever(search_kwargs={"k": 20})
query = "总统在关于Ketanji Brown Jackson的讲话中说了什么"
docs = retriever.invoke(query)
pretty_print_docs(docs)
```

```output
Document 1:
一个总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
4天前，我提名了联邦上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律智囊，将延续布雷耶法官的卓越传统。
----------------------------------------------------------------------------------------------------
Document 2:
正如我去年所说，特别是对我们年轻的跨性别美国人，作为总统，我将永远支持你们，让你们做自己，实现上帝赋予你们的潜力。
尽管我们似乎从不达成一致，但事实并非如此。去年，我签署了80项两党议案。从防止政府停摆到保护亚裔免受依然普遍的仇恨犯罪，再到改革军事司法。
----------------------------------------------------------------------------------------------------
Document 3:
曾在私人执业中担任高级诉讼律师。曾担任联邦公共辩护律师。来自公立学校教育工作者和警察家庭。一个建立共识的人。自提名以来，她得到了广泛支持——从警察工会到民主党和共和党任命的前法官。
如果我们要推进自由和正义，我们需要确保边境安全并修复移民制度。
----------------------------------------------------------------------------------------------------
Document 4:
他会见了乌克兰人民。
从泽连斯基总统到每一位乌克兰人，他们的无畏、勇气和决心激励着世界。
公民团体用自己的身体阻挡坦克。从学生到退休教师变成保卫家园的士兵。
正如泽连斯基总统在向欧洲议会发表的讲话中所说的那样，“光明将战胜黑暗。”乌克兰驻美国大使今晚在这里。
----------------------------------------------------------------------------------------------------
Document 5:
但这种滴灌理论导致经济增长疲弱、工资下降、赤字增大，以及近一个世纪以来最广泛的贫富差距。
哈里斯副总统和我竞选时提出了一个新的美国经济愿景。
投资于美国。教育美国人。培养劳动力。从底层和中间向上建设经济，而不是自上而下。
----------------------------------------------------------------------------------------------------
Document 6:
今晚，我宣布司法部将任命一名大流行欺诈首席检察官。
到今年年底，赤字将降至我上任前的不到一半。
是唯一一个在一年内削减赤字超过一万亿美元的总统。
降低您的成本还意味着要求更多的竞争。
我是资本家，但没有竞争的资本主义不是资本主义。
这是剥削，它推高价格。
----------------------------------------------------------------------------------------------------
Document 7:
我与他们的家人交谈，并告诉他们，我们永远感激他们的牺牲，我们将继续履行他们的使命，恢复每个社区应有的信任和安全。
我长期致力于这些问题。
我知道什么是有效的：投资于犯罪预防和社区警察，他们会巡逻，了解社区，并能恢复信任和安全。
因此，让我们不要放弃我们的街道。或者在安全和平等正义之间做选择。
----------------------------------------------------------------------------------------------------
Document 8:
正如我告诉习近平的，押注美国人民从来不是一个明智的选择。
我们将为数百万美国人创造就业机会，现代化美国各地的道路、机场、港口和水路。
我们将做到这一切，以抵御气候危机的破坏性影响，并促进环境正义。
----------------------------------------------------------------------------------------------------
Document 9:
议长女士，副总统女士，我们的第一夫人和第二先生。国会议员和内阁成员。最高法院法官。我的美国同胞。
去年，COVID-19让我们分开。今年，我们终于再次聚在一起。
今晚，我们作为民主党人、共和党人和独立人士相聚。但更重要的是作为美国人。
我们有责任对彼此、对美国人民、对宪法负责。
并怀着坚定的决心，自由将永远战胜暴政。
----------------------------------------------------------------------------------------------------
Document 10:
正如俄亥俄州参议员谢罗德·布朗所说，“是时候埋葬‘锈带’这个标签了。”
是时候了。
但尽管我们经济中有许多亮点，创纪录的就业增长和更高的工资，但太多家庭仍在努力跟上账单。
通货膨胀正在剥夺他们本应感受到的收益。
我明白。这就是为什么我的首要任务是控制价格。
----------------------------------------------------------------------------------------------------
Document 11:
我还呼吁国会：通过一项法律，确保在伊拉克和阿富汗受有毒暴露影响的退伍军人最终获得应得的福利和全面的医疗保健。
第四，让我们结束我们所知道的癌症。
这对我、吉尔、卡玛拉以及你们许多人来说都是个人问题。
癌症是美国的第二大死因，仅次于心脏病。
----------------------------------------------------------------------------------------------------
Document 12:
头痛。麻木。头晕。
一种癌症会让他们躺在用国旗覆盖的棺材里。
我知道。
其中一名士兵是我的儿子贝奥少校。我们不确定烧毁坑是否导致了他的脑癌，或者导致我们许多军人患病。但我承诺尽一切努力找出我们能找到的一切。
致力于像俄亥俄州的丹尼尔·罗宾逊这样的军事家庭。
他是希斯·罗宾逊一等兵的遗孀。
----------------------------------------------------------------------------------------------------
Document 13:
他永远不会扼杀他们对自由的热爱。他永远不会削弱自由世界的决心。
我们今晚聚集在一个经历过这个国家有史以来最艰难两年的美国。
这场大流行是惩罚性的。
许多家庭每月靠工资度日，努力跟上食品、汽油、住房等成本的上涨。
我理解。
----------------------------------------------------------------------------------------------------
Document 14:
当我们投资于我们的工人，当我们共同从底层和中间开始建设经济，我们可以做到我们很久没有做到的事情：建设一个更美好的美国。
两年多来，COVID-19影响了我们生活中的每一个决定，也影响了国家的生活。
我知道你们感到疲倦、沮丧和精疲力竭。
但我也知道这一点。
----------------------------------------------------------------------------------------------------
Document 15:
我的对抗通货膨胀计划将降低您的成本，降低赤字。
17位经济学诺贝尔奖获得者表示，我的计划将缓解长期通货膨胀压力。顶尖商界领袖和大多数美国人支持我的计划。以下是计划：
首先——降低处方药费用。只需看看胰岛素。十分之一的美国人患有糖尿病。在弗吉尼亚州，我遇到了一个名叫乔舒亚·戴维斯的13岁男孩。
----------------------------------------------------------------------------------------------------
Document 16:
很快，我们将加强我三十年前首次起草的《打击针对妇女暴力法》。向国家展示我们可以团结一致，做出重大举措。
因此，今晚我提出了一个国家团结议程。我们可以共同做的四件大事。
首先，战胜阿片类药物危机。
我们可以做很多事情。增加预防、治疗、危害减少和康复的资金。
----------------------------------------------------------------------------------------------------
Document 17:
我的计划不仅将降低成本，给家庭一个公平机会，还将降低赤字。
前一届政府不仅通过为富人和公司减税膨胀了赤字，还削弱了监督机构的职能，这些机构的职责是防止疫情救助资金被浪费。
但在我的政府中，监督机构已经受到欢迎。我们正在追捕那些窃取旨在帮助小企业和数百万美国人的援助资金的罪犯。
----------------------------------------------------------------------------------------------------
Document 18:
因此，让我们不要放弃我们的街道。或者在安全和平等正义之间做选择。
让我们团结起来保护我们的社区，恢复信任，并要求执法机构承担责任。
这就是为什么司法部要求其官员佩戴身体摄像头，禁止铐颈，限制无敲门搜查令。
----------------------------------------------------------------------------------------------------
Document 19:
我理解。
我记得我父亲不得不离开我们在宾夕法尼亚州斯克兰顿的家去找工作。我在一个家庭中长大，如果食品价格上涨，你会感受到。
这就是为什么我担任总统后的第一件事就是努力通过《美国复苏计划》。因为人们受到了伤害。我们需要采取行动，我们也做到了。
在我们历史上的关键时刻，几乎没有哪一项立法比这更多地帮助我们摆脱危机。
----------------------------------------------------------------------------------------------------
Document 20:
我们将作为一个民族。
一个美国。
美利坚合众国。
愿上帝保佑你们所有人。愿上帝保护我们的军队。
```

## 使用 LLMLingua 进行压缩

现在让我们用 `ContextualCompressionRetriever` 将我们的基础检索器包装起来，使用 `LLMLinguaCompressor` 作为压缩器。

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain_community.document_compressors import LLMLinguaCompressor
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0)
compressor = LLMLinguaCompressor(model_name="openai-community/gpt2", device_map="cpu")
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, base_retriever=retriever
)
compressed_docs = compression_retriever.invoke(
    "What did the president say about Ketanji Jackson Brown"
)
pretty_print_docs(compressed_docs)
```

```output
Document 1:
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
----------------------------------------------------------------------------------------------------
Document 2:
. Numbness. Dizziness.A that would them in a-draped coffin. I One of those soldiers was my Biden We don’t know for sure if a burn pit the cause of brain, or the diseases of so many of our troops But I’m committed to finding out everything we can Committed to military families like Danielle Robinson from Ohio The widow of First Robinson.
----------------------------------------------------------------------------------------------------
Document 3:
<ref#> let� Or between equal Let’ to protect, restore law accountable  why the Justice Department cameras bannedhold and restricted its officers. <
----------------------------------------------------------------------------------------------------
Document 4:
<# The Sergeant Class Combat froms widow us toBut burn pits ravaged Heath’s lungs and body. 
Danielle says Heath was a fighter to the very end.
```

## 使用 LLMLingua 进行 QA 生成

现在我们可以看看在生成步骤中使用它是什么样子的。

```python
from langchain.chains import RetrievalQA
chain = RetrievalQA.from_chain_type(llm=llm, retriever=compression_retriever)
```

```python
chain.invoke({"query": query})
```

```output
{'query': 'What did the president say about Ketanji Brown Jackson',
 'result': "The President mentioned that Ketanji Brown Jackson is one of the nation's top legal minds and will continue Justice Breyer's legacy of excellence."}
```