# 维基百科

[维基百科](https://wikipedia.org/)是一个多语种的免费在线百科全书，由一群志愿者社区（称为维基人）通过开放协作并使用名为MediaWiki的基于wiki的编辑系统编写和维护。`维基百科`是有史以来最大且最受欢迎的参考作品。

这个笔记本展示了如何从`wikipedia.org`检索维基页面并将其转换为下游使用的文档格式。

## 安装

首先，您需要安装`wikipedia` python包。

```python
%pip install --upgrade --quiet  wikipedia
```

`WikipediaRetriever`有以下参数：

- 可选参数`lang`：默认值为"en"。用于在维基百科的特定语言部分中进行搜索

- 可选参数`load_max_docs`：默认值为100。用于限制下载的文档数量。下载所有100个文档需要时间，因此在实验中使用较小的数字。目前有一个硬性限制为300。

- 可选参数`load_all_available_meta`：默认值为False。默认情况下，只下载最重要的字段：`Published`（文档发布/最后更新日期）、`title`、`Summary`。如果为True，则还会下载其他字段。

`get_relevant_documents()`有一个参数`query`：用于在维基百科中查找文档的自由文本。

## 示例

### 运行检索器

```python
from langchain_community.retrievers import WikipediaRetriever
```

```python
retriever = WikipediaRetriever()
```

```python
docs = retriever.invoke("HUNTER X HUNTER")
```

```python
docs[0].metadata  # 文档的元信息
```

```output
{'title': 'Hunter × Hunter',
 'summary': 'Hunter × Hunter（风格化为HUNTER×HUNTER，发音为“hunter hunter”）是由冈田斗司郎创作的日本漫画系列。自1998年3月以来，它一直在集英社的少年漫画杂志《周刊少年Jump》上连载，尽管自2006年以来，漫画经常会出现长时间的停刊。截至2022年11月，其章节已被收录在37卷的单行本中。故事聚焦于一个名叫小杰的年轻男孩，他发现自己的父亲在他很小的时候离开了他，实际上是一位世界知名的猎人，一名持牌专业人士，专门从事寻找稀有或未知动物物种、寻宝、勘测未开发的飞地或追捕无法无天的个体等幻想追求。小杰踏上了成为一名猎人并最终找到他的父亲的旅程。在此过程中，小杰遇到了其他各种猎人并接触到了超自然现象。Hunter × Hunter被改编为由日本动画制作公司日本动画制作的62集动画电视系列，由古桥一浩执导，于1999年10月至2001年3月在富士电视台播出。随后，日本动画制作公司制作了三部共30集的独立原创视频动画（OVA），并于2002年至2004年在日本发行。Madhouse制作的第二部动画电视系列于2011年10月至2014年9月在日本电视台播出，共148集，2013年还发布了两部动画剧场版电影。此外，还有许多基于Hunter × Hunter的音频专辑、视频游戏、音乐剧和其他媒体。这部漫画已被翻译成英文，并自2005年4月起由Viz Media在北美发行。两部电视系列也被Viz Media授权，第一部系列于2009年在Funimation频道播出，第二部系列于2016年4月至2019年6月在Adult Swim的Toonami节目区播出。Hunter × Hunter取得了巨大的批评和经济成功，并成为有史以来销量最高的漫画系列之一，截至2022年7月，其发行量已超过8400万册。'}
```

```python
docs[0].page_content[:400]  # 文档内容
```

```output
'Hunter × Hunter（风格化为HUNTER×HUNTER，发音为“hunter hunter”）是由冈田斗司郎创作的日本漫画系列。自1998年3月以来，它一直在集英社的少年漫画杂志《周刊少年Jump》上连载，尽管自2006年以来，漫画经常会出现长时间的停刊。截至2022年11月，其章节已被收录在37卷的单行本中。故事聚焦于一个名叫小杰的年轻男孩，他发现自己的父亲在他很小的时候离开了他，实际上是一位世界知名的猎人，一名持牌专业人士，专门从事寻找稀有或未知动物物种、寻宝、勘测未开发的飞地或追捕无法无天的个体等幻想追求。小杰踏上了成为一名猎人并最终找到他的父亲的旅程。在此过程中，小杰遇到了其他各种猎人并接触到了超自然现象。Hunter × Hunter被改编为由日本动画制作公司日本动画制作的62集动画电视系列，由古桥一浩执导，于1999年10月至2001年3月在富士电视台播出。随后，日本动画制作公司制作了三部共30集的独立原创视频动画（OVA），并于2002年至2004年在日本发行。Madhouse制作的第二部动画电视系列于2011年10月至2014年9月在日本电视台播出，共148集，2013年还发布了两部动画剧场版电影。此外，还有许多基于Hunter × Hunter的音频专辑、视频游戏、音乐剧和其他媒体。这部漫画已被翻译成英文，并自2005年4月起由Viz Media在北美发行。两部电视系列也被Viz Media授权，第一部系列于2009年在Funimation频道播出，第二部系列于2016年4月至2019年6月在Adult Swim的Toonami节目区播出。Hunter × Hunter取得了巨大的批评和经济成功，并成为有史以来销量最高的漫画系列之一，截至2022年7月，其发行量已超过8400万册。'
```

### 事实问答

```python
# 获取一个令牌：https://platform.openai.com/account/api-keys
from getpass import getpass
OPENAI_API_KEY = getpass()
```

```python
import os
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-3.5-turbo")  # 切换到'gpt-4'
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```

```python
questions = [
    "What is Apify?",
    "When the Monument to the Martyrs of the 1830 Revolution was created?",
    "What is the Abhayagiri Vihāra?",
    # "How big is Wikipédia en français?",
]
chat_history = []
for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **Question**: {question} \n")
    print(f"**Answer**: {result['answer']} \n")
```

```markdown
-> **问题**：Apify 是什么？
**答案**：Apify 是一个平台，可以让你轻松地自动化网页抓取、数据提取和网页自动化。它提供了一个基于云的基础设施，用于运行网络爬虫和其他自动化任务，以及一个基于网络的工具，用于构建和管理你的爬虫。此外，Apify 还提供了一个市场，用于购买和出售预先构建的爬虫和相关服务。
-> **问题**：1830年革命烈士纪念碑是什么时候创建的？
**答案**：Apify 是一个网页抓取和自动化平台，可以帮助你从网站上提取数据，将非结构化数据转化为结构化数据，并自动化重复性任务。它提供了一个用户友好的界面，用于创建网页抓取脚本，无需任何编码知识。Apify 可用于各种网页抓取任务，如数据提取、网页监控、内容聚合等。此外，它还提供了各种功能，如代理支持、调度和与其他工具的集成，以使网页抓取和自动化任务更加简单高效。
-> **问题**：阿帕耶吉利寺是什么？
**答案**：阿帕耶吉利寺是斯里兰卡阿努拉德普勒的南传佛教重要寺院遗址，建立于公元前2世纪，被认为是斯里兰卡最重要的僧院群之一。
```