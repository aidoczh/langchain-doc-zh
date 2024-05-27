# Exa 搜索

Exa（前身为 Metaphor Search）是一款专为 LLMs 设计的搜索引擎，可以使用**自然语言查询**在互联网上搜索文档，然后从所需文档中检索**清理过的 HTML 内容**。

与基于关键词的搜索（Google）不同，Exa 的神经搜索功能使其能够语义理解查询并返回相关文档。例如，我们可以搜索 `"关于猫的迷人文章"`，并比较来自[Google](https://www.google.com/search?q=fascinating+article+about+cats)和[Exa](https://search.exa.ai/search?q=fascinating%20article%20about%20cats&autopromptString=Here%20is%20a%20fascinating%20article%20about%20cats%3A)的搜索结果。Google 给出了基于关键词“迷人”的经过 SEO 优化的列表文章。Exa 则直接奏效。

本笔记介绍了如何使用 LangChain 进行 Exa 搜索。

首先，获取 Exa API 密钥并将其添加为环境变量。通过[在此处注册](https://dashboard.exa.ai/)，每月可获得 1000 次免费搜索。

```python
import os
os.environ["EXA_API_KEY"] = "..."
```

然后安装集成包

```python
%pip install --upgrade --quiet langchain-exa
# 还有一些此笔记所需的依赖包
%pip install --upgrade --quiet langchain langchain-openai
```

## 使用 ExaSearchRetriever

ExaSearchRetriever 是一个使用 Exa 搜索检索相关文档的检索器。

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_exa import ExaSearchRetriever, TextContentsOptions
from langchain_openai import ChatOpenAI
# 检索 5 个文档，内容截断为 1000 个字符
retriever = ExaSearchRetriever(
    k=5, text_contents_options=TextContentsOptions(max_length=200)
)
prompt = PromptTemplate.from_template(
    """Answer the following query based on the following context:
query: {query}
<context>
{context}
</context"""
)
llm = ChatOpenAI()
chain = (
    RunnableParallel({"context": retriever, "query": RunnablePassthrough()})
    | prompt
    | llm
)
chain.invoke("When is the best time to visit japan?")
```

```output
[Result(title='Find Us:', url='https://travelila.com/best-time-to-visit-japan/', id='UFLQGtanQffaDErhngnzgA', score=0.1865834891796112, published_date='2021-01-05', author=None, text='If you are planning to spend your next vacation in Japan, then hold your excitement a bit. It would help if you planned which places you will visit in Japan and the country’s best things. It’s entirel', highlights=None, highlight_scores=None), Result(title='When Is The Best Time of Year To Visit Japan?', url='https://boutiquejapan.com/when-is-the-best-time-of-year-to-visit-japan/', id='70b0IMuaQpshjpBpnwsfUg', score=0.17796635627746582, published_date='2022-09-26', author='Andres Zuleta', text='The good news for travelers is that there is no single best time of year to travel to Japan — yet this makes it hard to decide when to visit, as each season has its own special highlights.When plannin', highlights=None, highlight_scores=None), Result(title='Here is the Best Time to Visit Japan - Cooking Sun', url='https://www.cooking-sun.com/best-time-to-visit-japan/', id='2mh-xvoqGPT-ZRvX9GezNQ', score=0.17497511208057404, published_date='2018-12-17', author='Cooking Sun', text='Japan is a diverse and beautiful country that’s brimming with culture. For some travelers, visiting Japan is a dream come true, since it grazes bucket lists across the globe. One of the best parts abo', highlights=None, highlight_scores=None), Result(title='When to Visit Japan? Bests Times and 2023 Travel Tips', url='https://www.jrailpass.com/blog/when-visit-japan-times', id='KqCnY8fF-nc76n1wNpIo1Q', score=0.17359933257102966, published_date='2020-02-18', author='JRailPass', text='When is the best time to visit Japan? This is a question without a simple answer. Japan is a year-round destination, with interesting activities, attractions, and festivities throughout the year.Your ', highlights=None, highlight_scores=None), Result(title='Complete Guide To Visiting Japan In February 2023: Weather, What To See & Do | LIVE JAPAN travel guide', url='https://livejapan.com/en/article-a0002948/', id='i3nmekOdM8_VBxPfcJmxng', score=0.17215865850448608, published_date='2019-11-13', author='Lucio Maurizi', text='\n \n \n HOME\n Complete Guide To Visiting Japan In February 2023: Weather, What To See & Do\n \n \n \n \n \n \n Date published: 13 November 2019 \n Last updated: 26 January 2021 \n \n \n So you’re planning your tra', highlights=None, highlight_scores=None)]
```

```output
AIMessage(content='Based on the given context, there is no specific best time mentioned to visit Japan. Each season has its own special highlights, and Japan is a year-round destination with interesting activities, attractions, and festivities throughout the year. Therefore, the best time to visit Japan depends on personal preferences and the specific activities or events one wants to experience.')
```

## 使用 Exa SDK 作为 LangChain Agent 工具

[Exa SDK](https://docs.exa.ai/) 创建了一个客户端，可以使用 Exa API 执行三个功能：

- `search`：根据自然语言搜索查询，检索搜索结果列表。

- `find_similar`：根据 URL，检索与提供的 URL 对应的相似网页的搜索结果列表。

- `get_content`：根据从 `search` 或 `find_similar` 获取的文档 ID 列表，获取每个文档的清理后的 HTML 内容。

我们可以使用 `@tool` 装饰器和文档字符串创建 LangChain 工具包装器，告诉 LLM Agent 如何使用 Exa。

```python
%pip install --upgrade --quiet  langchain-exa
```

```python
from exa_py import Exa
from langchain_core.tools import tool
exa = Exa(api_key=os.environ["EXA_API_KEY"])
@tool
def search(query: str):
    """根据查询搜索网页。"""
    return exa.search(f"{query}", use_autoprompt=True, num_results=5)
@tool
def find_similar(url: str):
    """搜索与给定 URL 相似的网页。
    传入的 URL 应该是从 `search` 返回的 URL。
    """
    return exa.find_similar(url, num_results=5)
@tool
def get_contents(ids: list[str]):
    """获取网页的内容。
    传入的 ids 应该是从 `search` 返回的 ID 列表。
    """
    return exa.get_contents(ids)
tools = [search, get_contents, find_similar]
```

### 提供 Exa 工具给 Agent

我们可以将刚刚创建的 Exa 工具提供给 LangChain `OpenAIFunctionsAgent`。当要求“为我总结一篇关于猫的有趣文章”时，Agent 使用 `search` 工具执行 Exa 搜索，使用 `get_contents` 工具执行 Exa 内容检索，然后返回检索内容的摘要。

```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0)
system_message = SystemMessage(
    content="You are a web researcher who answers user questions by looking up information on the internet and retrieving contents of helpful documents. Cite your sources."
)
agent_prompt = OpenAIFunctionsAgent.create_prompt(system_message)
agent = OpenAIFunctionsAgent(llm=llm, tools=tools, prompt=agent_prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

```python
agent_executor.run("Summarize for me a fascinating article about cats.")
```

```output
> 进入新的 AgentExecutor 链...
调用：`search`，参数为 `{'query': 'fascinating article about cats'}`
标题：How Much Would You Pay to Save Your Cat’s Life?
URL：https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/
ID：8uIICapOJD68i8iKEaRfPg
得分：0.17565323412418365
发布日期：2022-11-20
作者：Sarah Zhang
摘要：无
标题：How Your Cat Is Making You Crazy
URL：https://www.theatlantic.com/magazine/archive/2012/03/how-your-cat-is-making-you-crazy/308873/
ID：tqgaWIm4JkU9wfINfE-52Q
得分：0.1744839996099472
发布日期：2012-02-06
作者：Kathleen McAuliffe
摘要：无
标题：The Feline Mystique
URL：https://www.mcsweeneys.net/articles/the-feline-mystique
ID：wq8uMC5BEjSmvA-NBSCTvQ
得分：0.17415249347686768
发布日期：2022-07-19
作者：Kathryn Baecht
摘要：无
标题：How Much Would You Pay to Save Your Cat’s Life?
URL：https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/?utm%5C_source=pocket-newtab-global-en-GB
ID：wpT7Ee9vkcJyDUHgOelD_w
得分：0.17202475666999817
发布日期：2022-11-20
作者：Sarah Zhang
摘要：无
标题：How Much Would You Pay to Save Your Cat’s Life?
URL：https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/?src=longreads
ID：IkNF5fN0F8B0W_-Boh567A
得分：0.16996535658836365
发布日期：2022-11-20
作者：Sarah Zhang
摘要：无
自动提示字符串：这是一篇关于猫的有趣文章：
调用：`get_contents`，参数为 `{'ids': ['8uIICapOJD68i8iKEaRfPg']}`
ID：8uIICapOJD68i8iKEaRfPg
URL：https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/
标题：How Much Would You Pay to Save Your Cat’s Life?
```

```markdown
# 提取：  
<div><article><header><div><div><p>你可以花15000美元给你的宠物做肾移植手术。</p><div><address>作者：<a href="https://www.theatlantic.com/author/sarah-zhang/">Sarah Zhang</a><p>摄影：<i>The Atlantic</i> Caroline Tompkins</p></address></div></div><div><figure><div></div><figcaption>夏洛克在2019年捐献了一颗肾。（Caroline Tompkins for The Atlantic）</figcaption></figure></div></div></header><section><figure></figure><p><small><i>本文曾刊登在《每日一读》（One Story to Read Today）中，这是我们编辑每周一至周五推荐的必读文章的新闻简报。</i><a href="https://www.theatlantic.com/newsletters/sign-up/one-story-to-read-today/"><i>在此订阅。</i></a><i>      </i></small></p><p>当我第一次见到16岁的草莓时，她躺在地上，四肢伸展。她的猫肚被剃得精光，黑色的线缝了几英寸长，贯穿她裸露的粉红皮肤。</p><p>一名放射科医生在她的腹部涂抹超声波凝胶，而两名穿着深蓝色外科服的兽医学生轻轻按住她的腿——尽管这实际上并不是必要的。草莓太累了，麻醉药效太大，或者仅仅是因为前一天的手术让她神志不清。在放射室昏暗的灯光下，她的瞳孔扩大成深黑色的池塘。她慢慢地转过头看着我。她又转开了。她环顾了一下围绕她的医生和学生小群，仿佛在想弄清楚究竟发生了什么让她陷入这种境地。</p><section><figure><a href="https://www.theatlantic.com/magazine/toc/2022/12/"></a></figure><div><h2>探索2022年12月刊</h2><p>查看更多内容并找到下一篇要读的文章。</p></div><a href="https://www.theatlantic.com/magazine/toc/2022/12/">查看更多</a></section><p>发生的事情是，草莓接受了肾移植手术。佐治亚大学的一个外科团队剃掉了她长长的姜色毛发，插入导管到她的腿和颈部，以输送她在医院期间所需的药物组合：麻醉药、止痛药、抗生素、抗凝血剂和免疫抑制剂。然后，一位名叫查德·施密特（Chad Schmiedt）的外科医生小心翼翼地切开她的腹部中线——经过两个已经不再起作用的干枯肾脏，几乎到她的腹股沟。接下来，他将一颗健康的新肾缝合到位，这颗肾是几个小时前从活体捐赠者那里新鲜取出的。</p><p>施密特是为数不多的进行猫儿移植手术的外科医生之一，因此也是世界上最权威的连接猫儿肾脏的专家之一。当他用宽厚的微笑和握手迎接我的时候，我被他那双大而粗糙的手所震撼。然而，在手术室里，他的手却以微观精度工作，缝合只有毫米宽的动脉和静脉。他告诉我，这是最困难的部分，就像缝“湿米纸”一样。一旦捐赠者的肾脏就位，它就变得粉红色，施密特将草莓缝合好。 （和人类移植手术一样，旧肾可以留在原位。）接下来就是等待她醒来和排尿。在她做超声检查时，她已经做到了这两点。</p><p>草莓并不能理解这一切——也没有任何猫能理解为什么我们人类坚持要把它们带到兽医诊所让陌生人来戳来捅。但如果没有移植手术，她将死于肾衰竭，这种病症类似于逐渐被内部毒害。其他治疗方法可以减缓她的肾病，这在老年猫中很常见，但无法阻止病情恶化。这就是为什么草莓的主人决定花15000美元做肾移植手术——这是拯救她生命的最后手段，或者至少是延长她的生命。</p><p>那天我没有在医院见到她的主人。草莓需要至少住院一周。作者：无。这是一篇由Sarah Zhang撰写的关于猫的引人入胜的文章，题为《你愿意为了拯救你的猫花多少钱？》。文章讨论了宠物的重症护理成本以及一些主人为拯救他们的猫而愿意付出的努力。文章重点介绍了一只名叫草莓的猫接受了肾移植手术，这是由少数专门从事猫儿移植手术的外科医生之一进行的。文章探讨了提供拯救宠物生命的治疗所涉及的情感和财务决策。您可以在[这里](https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/)阅读全文。
> 完成链。
```

这是一篇由Sarah Zhang撰写的关于猫的有趣文章，题为《为了拯救你的猫，你愿意付出多少代价？》。文章讨论了宠物紧急护理的费用以及一些主人为拯救猫的生命而愿意付出的努力。文章提到了一只名叫Strawberry的猫接受了肾移植手术，这是由少数专门从事猫移植手术的外科医生进行的。文章探讨了为宠物提供挽救生命治疗所涉及的情感和财务决策。你可以在[这里](https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/)阅读全文。

## 高级Exa功能

Exa支持按领域和日期进行强大的过滤。我们可以为代理提供更强大的`search`工具，让它决定是否应用过滤器，如果这些过滤器对目标有用的话。在[这里](https://github.com/metaphorsystems/metaphor-python/)查看Exa的所有搜索功能。

```python
from exa_py import Exa
from langchain_core.tools import tool
exa = Exa(api_key=os.environ["Exa_API_KEY"])
@tool
def search(query: str, include_domains=None, start_published_date=None):
    """根据查询搜索网页。
    将可选的include_domains（list[str]）参数设置为限制搜索到一组域名。
    将可选的start_published_date（str）参数设置为限制搜索到指定日期之后发布的文档（YYYY-MM-DD）。
    """
    return exa.search_and_contents(
        f"{query}",
        use_autoprompt=True,
        num_results=5,
        include_domains=include_domains,
        start_published_date=start_published_date,
    )
@tool
def find_similar(url: str):
    """搜索与给定URL相似的网页。
    传入的url应该是从`search`返回的URL。
    """
    return exa.find_similar_and_contents(url, num_results=5)
@tool
def get_contents(ids: list[str]):
    """获取网页的内容。
    传入的ids应该是从`search`返回的id列表。
    """
    return exa.get_contents(ids)
tools = [search, get_contents, find_similar]
```

现在我们要求代理根据领域和发布日期的限制对一篇文章进行摘要。我们将使用GPT-4代理，以获得更强大的推理能力，以支持更复杂的工具使用。

代理正确地使用搜索过滤器找到了符合所需限制的文章，并再次检索内容并返回摘要。

```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0, model="gpt-4")
system_message = SystemMessage(
    content="你是一位网络研究员，通过查找互联网上的信息并检索有用文档的内容来回答用户的问题。引用你的来源。"
)
agent_prompt = OpenAIFunctionsAgent.create_prompt(system_message)
agent = OpenAIFunctionsAgent(llm=llm, tools=tools, prompt=agent_prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

```python
agent_executor.run(
    "为我总结一篇有关AI的有趣文章，来自lesswrong.com，并且发布日期在2023年10月之后。"
)
```

```output

> 进入新的AgentExecutor链...

调用：`search`，参数为`{'query': 'AI article', 'include_domains': ['lesswrong.com'], 'start_published_date': '2023-10-01'}`

标题：A dialectical view of the history of AI, Part 1: We’re only in the antithesis phase. [A synthesis is in the future.]

URL：https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in

ID：45-Cs6bdV4_HB8epTO0utg

得分：0.14216266572475433

发布日期：2023-11-16

作者：Bill Benzon

摘要：无

标题：Possible OpenAI's Q* breakthrough and DeepMind's AlphaGo-type systems plus LLMs

URL：https://www.lesswrong.com/posts/JnM3EHegiBePeKkLc/possible-openai-s-q-breakthrough-and-deepmind-s-alphago-type

ID：BPJkDFi_bcpo7T19zlj0Ag

得分：0.13007280230522156

发布日期：2023-11-23

作者：Burny

摘要：无

标题：Architects of Our Own Demise: We Should Stop Developing AI

URL：https://www.lesswrong.com/posts/bHHrdXwrCj2LRa2sW/architects-of-our-own-demise-we-should-stop-developing-ai

ID：IMLGjczHCLOuoB6w_vemkA

得分：0.12803198397159576

发布日期：2023-10-26

作者：Roko

摘要：无

标题：Benevolent [ie, Ruler] AI is a bad idea and a suggested alternative

URL：https://www.lesswrong.com/posts/bwL63TrCo4RaHc3WJ/benevolent-ie-ruler-ai-is-a-bad-idea-and-a-suggested

ID：RvRnTH2cpV186d98SFbb9w

得分：0.12487411499023438

发布日期：2023-11-19

作者：The Gears

摘要：无

标题：Benevolent [Ruler] AI is a bad idea and a suggested alternative

链接: [原文链接](https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in)

标题: 人工智能历史的辩证观，第一部分：我们只处于反对阶段。[综合将来会出现]

作者: The Gears

发表日期: 2023年11月19日

提取: 无

自动提示字符串: 这是一篇关于人工智能的有趣文章：

调用: 使用 `{'ids': ['45-Cs6bdV4_HB8epTO0utg']}` 参数调用 `get_contents`

链接: [原文链接](https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in)

标题: 人工智能历史的辩证观，第一部分：我们只处于反对阶段。[综合将来会出现]

提取: <div><div><p><i>转载自 </i><a href="https://new-savanna.blogspot.com/2023/11/a-dialectical-view-of-history-of-ai.html"><i>新荒野</i></a><i>。</i></p><p>认为<a href="https://www.britannica.com/topic/philosophy-of-history/History-as-a-process-of-dialectical-change-Hegel-and-Marx">历史是通过辩证变化进行的</a>这一观念主要归功于黑格尔和马克思。虽然我在职业生涯早期读过他们的一些著作，但并没有受到深刻影响。尽管如此，我发现历史通过辩证变化进行的观念是一种有用的思考人工智能历史的方式。因为这意味着历史不仅仅是一件事情或另一件事情。</p><p>这种辩证过程通常被概括为从一个命题到一个反命题，最终到一个“更高层次”的综合的运动，不管那是什么。技术术语是<i>Aufhebung</i>。维基百科：</p><blockquote><p>在黑格尔那里，<i>Aufhebung</i>这个术语表面上有保留和改变的矛盾含义，最终是进步（德语动词<i>aufheben</i>的意思是“取消”，“保留”和“拾起”）。这些意义之间的张力适合黑格尔所要谈论的内容。在升华中，一个术语或概念通过其与另一个术语或概念的辩证相互作用而既被保留又被改变。升华是辩证法运作的动力。</p></blockquote><p>那么，为什么我认为人工智能历史最好以这种方式构想呢？第一个时代，命题，从1950年代一直延续到1980年代，基于自上而下的演绎<i>符号</i>方法。第二个时代，反命题，从1990年代开始上升，现在统治，基于自下而上的<i>统计</i>方法。这些在概念上和计算上是完全不同的，可以说是相反的。至于第三个时代，综合，嗯，我们甚至不知道是否会有第三个时代。也许第二个，当前的时代会带我们走<i>全部的路</i>，不管那意味着什么。我持怀疑态度。我相信会有第三个时代，它将涉及从前两个时代综合的概念思想和计算技术。</p><p>不过，需要注意的是，我将集中关注语言建模的工作。首先，那是我最擅长的。然而更重要的是，目前正引发对人工智能未来最狂热的猜测的是语言的工作。</p><p>让我们来看看。找个舒适的椅子，调整灯光，倒杯饮料，坐下，放松，阅读。这将需要一些时间。</p><p><strong>符号人工智能：命题</strong></p><p>人工智能的追求始于20世纪50年代，它始于某些思想和某些计算能力。后者粗糙，并且在今天的标准下极度不足。至于思想，我们需要两个或多或少独立的起点。一个给了我们“人工智能”（AI）这个术语，约翰·麦卡锡在1956年达特茅斯举办的一次会议上创造了这个词。另一个与机器翻译（MT）的追求有关，这在美国意味着将俄文技术文件翻译成英文。MT的目标是实际的，不断地实际的。没有谈论智能和图灵测试之类的东西。唯一重要的是能够拿俄文文本，输入计算机，得到一个能胜任的英文翻译。承诺已经做出，但几乎没有兑现。国防部在1960年代中期中止了这项工作。MT的研究人员随后重新打造自己，成为语言的研究者。作者：无此人名。在LessWrong上发表的题为“人工智能历史的辩证观，第一部分：我们只处于反对阶段。[综合将来会出现]”的文章提供了对人工智能演变的独特视角。作者Bill Benzon使用了主要归功于黑格尔和马克思的辩证变化概念来解释人工智能的历史。

这篇在 LessWrong 上的文章标题为《AI历史的辩证观，第一部分：我们只处于反对论阶段。【综合将在未来出现。】》，提供了对人工智能演变的独特视角。作者 Bill Benzon 使用了辩证变化的概念，主要归功于黑格尔和马克思，来解释人工智能的历史。

辩证过程被描述为从一个命题到一个反命题，最终到一个更高层次的综合的运动。作者将这一概念应用到人工智能上，确定了三个时代。第一个时代，即“命题”，从1950年代到1980年代，基于自上而下的演绎符号方法。第二个时代，即“反命题”，始于1990年代，基于自下而上的统计方法。这两个时代在概念上和计算上有相当大的不同。

至于第三个时代，即“综合”，作者推测它将涉及从前两个时代综合概念和计算技术。然而，目前尚不清楚这个时代是否会出现，或者当前时代是否会将我们“走到底”。

作者还指出，文章的重点是关于模拟语言的努力，因为目前关于人工智能未来最多的猜测是关于语言的工作。文章随后深入探讨了符号人工智能的历史，从20世纪50年代约翰·麦卡锡创造“人工智能”一词开始。

这篇文章全面而发人深省地展示了人工智能的历史和潜在未来，特别是在语言建模的背景下。

来源：[LessWrong](https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in)