# Vectara

[Vectara](https://vectara.com/) 是一款可信赖的 GenAI 平台，提供了一个易于使用的文档索引和查询的 API。

Vectara 提供了一种端到端的托管服务，用于检索增强生成（Retrieval Augmented Generation，RAG），其中包括：

1. 从文档文件中提取文本并将其分块为句子的方法。

2. 最先进的 [Boomerang](https://vectara.com/how-boomerang-takes-retrieval-augmented-generation-to-the-next-level-via-grounded-generation/) 嵌入模型。使用 Boomerang 将每个文本块编码为向量嵌入，并存储在 Vectara 的内部知识（向量+文本）存储中。

3. 一个查询服务，可以自动将查询编码为嵌入，并检索出最相关的文本片段（包括对 [混合搜索](https://docs.vectara.com/docs/api-reference/search-apis/lexical-matching) 和 [MMR](https://vectara.com/get-diverse-results-and-comprehensive-summaries-with-vectaras-mmr-reranker/) 的支持）。

4. 可选择创建基于检索到的文档的 [生成式摘要](https://docs.vectara.com/docs/learn/grounded-generation/grounded-generation-overview)，包括引用。

有关如何使用 API 的更多信息，请参阅 [Vectara API 文档](https://docs.vectara.com/docs/)。

本笔记本展示了如何使用与 `Vectara` 集成的 langchain 功能。具体来说，我们将演示如何使用 [LangChain 的表达式语言](/docs/concepts#langchain-expression-language) 和使用 Vectara 的集成摘要功能。

# 设置

您需要一个 Vectara 账户才能使用 Vectara 和 LangChain。要开始使用，请按照以下步骤操作：

1. 如果您还没有 Vectara 账户，请[注册](https://www.vectara.com/integrations/langchain)一个 Vectara 账户。完成注册后，您将获得一个 Vectara 客户 ID。您可以通过点击 Vectara 控制台窗口右上角的用户名来找到您的客户 ID。

2. 在您的账户中，您可以创建一个或多个语料库。每个语料库代表一个区域，用于存储从输入文档中摄取的文本数据。要创建一个语料库，请使用**"创建语料库"**按钮。然后，为您的语料库提供一个名称和描述。您还可以定义过滤属性并应用一些高级选项。如果您点击您创建的语料库，您可以在顶部看到它的名称和语料库 ID。

3. 接下来，您需要创建用于访问语料库的 API 密钥。在语料库视图中点击**"授权"**选项卡，然后点击**"创建 API 密钥"**按钮。给您的密钥取一个名称，并选择您希望密钥是仅查询还是查询+索引。点击**"创建"**，现在您有一个活动的 API 密钥。请保密此密钥。

要使用 Vectara 和 LangChain，您需要这三个值：客户 ID、语料库 ID 和 API 密钥。您可以通过以下两种方式将它们提供给 LangChain：

1. 在您的环境中包含这三个变量：`VECTARA_CUSTOMER_ID`、`VECTARA_CORPUS_ID` 和 `VECTARA_API_KEY`。

> 例如，您可以使用 os.environ 和 getpass 设置这些变量，如下所示：

```python
import os
import getpass
os.environ["VECTARA_CUSTOMER_ID"] = getpass.getpass("Vectara 客户 ID:")
os.environ["VECTARA_CORPUS_ID"] = getpass.getpass("Vectara 语料库 ID:")
os.environ["VECTARA_API_KEY"] = getpass.getpass("Vectara API 密钥:")
```

2. 将它们添加到 Vectara vectorstore 构造函数中：

```python
vectorstore = Vectara(
                vectara_customer_id=vectara_customer_id,
                vectara_corpus_id=vectara_corpus_id,
                vectara_api_key=vectara_api_key
            )
```

```python
from langchain_community.embeddings import FakeEmbeddings
from langchain_community.vectorstores import Vectara
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda, RunnablePassthrough
```

首先，我们将国情咨文文本加载到 Vectara 中。请注意，我们使用的是 `from_files` 接口，它不需要任何本地处理或分块 - Vectara 接收文件内容并执行所有必要的预处理、分块和嵌入操作，将文件嵌入到其知识存储中。

```python
vectara = Vectara.from_files(["state_of_the_union.txt"])
```

现在，我们创建一个 Vectara 检索器，并指定：

* 它只返回前 3 个匹配的文档

* 对于摘要，它应使用前 5 个结果，并用英语回复

```python
summary_config = {"is_enabled": True, "max_results": 5, "response_lang": "eng"}
retriever = vectara.as_retriever(
    search_kwargs={"k": 3, "summary_config": summary_config}
)
```

在使用 Vectara 进行摘要时，检索器将以 `Document` 对象列表的形式响应：

1. 前 `k` 个文档是与查询匹配的文档（与标准向量存储一样）

2. 在启用摘要功能后，会附加一个额外的`Document`对象，其中包含摘要文本。这个`Document`对象的元数据字段`summary`被设置为`True`。

让我们定义两个实用函数来分离它们：

```python
def get_sources(documents):
    return documents[:-1]
def get_summary(documents):
    return documents[-1].page_content
query_str = "拜登说了什么？"
```

现在我们可以尝试对查询进行摘要回答：

```python
(retriever | get_summary).invoke(query_str)
```

```output
'返回的结果不包含足够的信息，无法摘要成对您查询有用的答案。请尝试不同的搜索或重新陈述您的查询。'
```

如果我们想要查看从 Vectara 检索到的用于摘要的来源（引文）：

```python
(retriever | get_sources).invoke(query_str)
```

```output
[Document(page_content='当他们回家时，许多世界上最健壮、训练最好的战士再也不一样了。头晕。一个会让他们躺在披着国旗的棺材里的癌症。我知道。其中一位士兵是我儿子博伊中校。我们不确定燃烧坑是否是他的脑癌或我们的许多军队疾病的原因。但我致力于尽我们所能找出一切。', metadata={'lang': 'eng', 'section': '1', 'offset': '34652', 'len': '60', 'X-TIKA:Parsed-By': 'org.apache.tika.parser.csv.TextAndCSVParser', 'Content-Encoding': 'UTF-8', 'Content-Type': 'text/plain; charset=UTF-8', 'source': 'vectara'}),
 Document(page_content='美国司法部正在组建一个专门的特别行动小组，追查俄罗斯寡头的犯罪行为。我们将与我们的欧洲盟友合作，找到并没收你们的游艇、豪华公寓和私人飞机。我们要追回你们不义之财。今晚，我宣布我们将与盟友一起关闭美国领空对所有俄罗斯航班的开放，进一步孤立俄罗斯，并对他们的经济施加额外压力。卢布已贬值30%。', metadata={'lang': 'eng', 'section': '1', 'offset': '3807', 'len': '42', 'X-TIKA:Parsed-By': 'org.apache.tika.parser.csv.TextAndCSVParser', 'Content-Encoding': 'UTF-8', 'Content-Type': 'text/plain; charset=UTF-8', 'source': 'vectara'}),
 Document(page_content='他拒绝了多次外交努力。他认为西方和北约不会作出回应。他认为他可以在国内分裂我们。我们已经准备好了。以下是我们所做的。我们进行了广泛而仔细的准备。', metadata={'lang': 'eng', 'section': '1', 'offset': '2100', 'len': '42', 'X-TIKA:Parsed-By': 'org.apache.tika.parser.csv.TextAndCSVParser', 'Content-Encoding': 'UTF-8', 'Content-Type': 'text/plain; charset=UTF-8', 'source': 'vectara'})]
```

Vectara的“RAG服务”在创建问答或聊天机器人链时承担了很多繁重的工作。与LangChain的集成提供了使用其他功能的选项，例如查询预处理，如`SelfQueryRetriever`或`MultiQueryRetriever`。让我们看一个使用[MultiQueryRetriever](/docs/how_to/MultiQueryRetriever)的示例。

由于MQR使用了LLM，我们必须进行设置 - 这里我们选择了`ChatOpenAI`：

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0)
mqr = MultiQueryRetriever.from_llm(retriever=retriever, llm=llm)
(mqr | get_summary).invoke(query_str)
```

```output
"拜登总统发表了几个值得注意的引文和评论。他表达了调查燃烧坑对士兵健康的潜在影响的承诺，提到了他儿子的脑癌[1]。他强调了美国人团结的重要性，敦促我们将彼此视为同胞而不是敌人[2]。拜登还强调了学校应该使用美国复苏计划的资金来聘请教师和解决学习损失问题，同时鼓励社区参与支持教育[3]。"
```

```python
(mqr | get_sources).invoke(query_str)
```

```markdown
当他们回家时，世界上许多身体最健壮、训练最出色的战士已经不再是原来的自己。头晕目眩。
一种癌症让他们躺在用国旗裹着的棺材里。我知道。
我的儿子博伊·拜登少校就是其中之一。我们不确定燃烧坑是否导致了他的脑癌，或者导致了我们许多部队的疾病。但我承诺，我们会尽一切努力找出我们能找到的一切信息。[20]
美国司法部正在组建一个专门的特遣队，追查俄罗斯寡头的犯罪行为。我们将与我们的欧洲盟友合作，查找并没收你们的游艇、豪华公寓和私人飞机。我们将追缴你们非法获得的财富。今晚，我宣布我们将与盟友一道关闭美国领空，禁止所有俄罗斯航班——进一步孤立俄罗斯，并对他们的经济施加额外压力。卢布已贬值30%。[21]
而且，如果国会提供我们需要的资金，我们将准备好新的测试、口罩和药丸储备，以备不时之需。我不能保证不会出现新的变种。但我可以向你们保证，我们将尽一切努力做好准备。第三，我们可以结束学校和企业的关闭。我们已经拥有必要的工具。[22]
返回的结果并不包含足够的信息，无法总结成对您查询有用的答案。请尝试不同的搜索或以不同方式重述您的查询。
丹妮尔说希斯一直是一名战士，直到最后一刻。他不知道如何停止战斗，她也是如此。在痛苦中，她找到了要求我们做得更好的目的。今晚，丹妮尔——我们正在做到。退伍军人事务部正在开创将有毒暴露与疾病联系起来的新方法，已经帮助更多退伍军人获得福利。[23]
让我们停止把彼此视为敌人，开始看到彼此真正的样子：同胞。我们无法改变我们之间的分歧。但我们可以改变我们如何共同前进——在新冠肺炎和其他我们必须共同面对的问题上。不久前，我参观了纽约市警察局，在警官威尔伯特·莫拉和他的搭档杰森·里维拉的葬礼后的几天。他们在响应一个报警时，一名男子用一把偷来的枪射杀了他们。[24]
美国救援计划为学校提供了资金，以聘请教师并帮助学生弥补失去的学习时间。我敦促每个家长确保你的学校做到这一点。我们都可以发挥作用——注册成为导师或指导者。在大流行之前，儿童也在挣扎。欺凌、暴力、创伤和社交媒体的伤害。[25]
```

抱歉，我无法完成这个任务。