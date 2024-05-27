# AskNews

[AskNews](https://asknews.app) 通过单一的自然语言查询为任何LLM注入最新的全球新闻（或历史新闻）。具体来说，AskNews每天丰富超过30万篇文章，通过翻译、摘要、提取实体并将它们索引到热向量和冷向量数据库中。AskNews将这些向量数据库放在一个低延迟的端点上供您使用。当您查询AskNews时，您会收到一个经过优化的提示字符串，其中包含所有最相关的丰富信息（例如实体、分类、翻译、摘要）。这意味着您无需管理自己的新闻RAG，也无需担心如何以简洁的方式正确传达新闻信息给您的LLM。

AskNews还致力于透明度，这就是为什么我们的报道受到监控，并在数百个国家、13种语言和5万个来源之间进行了多样化。如果您想追踪我们的来源覆盖范围，您可以访问我们的[透明度仪表板](https://asknews.app/en/transparency)。

## 设置

集成位于`langchain-community`包中。我们还需要安装`asknews`包本身。

```bash
pip install -U langchain-community asknews
```

我们还需要设置我们的AskNews API凭据，这可以在[AskNews控制台](https://my.asknews.app)上获得。

```python
import getpass
import os
os.environ["ASKNEWS_CLIENT_ID"] = getpass.getpass()
os.environ["ASKNEWS_CLIENT_SECRET"] = getpass.getpass()
```

## 用法

这里我们展示如何单独使用这个工具。

```python
from langchain_community.tools.asknews import AskNewsSearch
tool = AskNewsSearch(max_results=2)
tool.invoke({"query": "Effect of fed policy on tech sector"})
```

```output
"\n<doc>\n[1]:\ntitle: Market Awaits Comments From Key Fed Official\nsummary: The market is awaiting comments from Fed Governor Christopher Waller, but it's not expected to move markets significantly. The recent Consumer Price Index (CPI) report showed slimming inflation figures, leading to a conclusion that inflation is being curbed. This has led to a 'soft landing' narrative, causing bullish sentiment in the stock market, with the Dow, S&P 500, Nasdaq, and Russell 2000 indices near all-time highs. NVIDIA is set to report earnings next week, and despite its 95% year-to-date growth, it remains a Zacks Rank #1 (Strong Buy) stock. The article also mentions upcoming economic data releases, including New and Existing Home Sales, S&P flash PMI Services and Manufacturing, Durable Goods, and Weekly Jobless Claims.\nsource: Yahoo\npublished: May 17 2024 14:53\nOrganization: Nasdaq, Fed, NVIDIA, Zacks\nPerson: Christopher Waller\nEvent: Weekly Jobless Claims\nclassification: Business\nsentiment: 0\n</doc>\n\n<doc>\n[2]:\ntitle: US futures flat as Fed comments cool rate cut optimism\nsummary: US stock index futures remained flat on Thursday evening, following a weak finish on Wall Street, as Federal Reserve officials warned that bets on interest rate cuts were potentially premature. The Fed officials, including Atlanta Fed President Raphael Bostic, New York Fed President John Williams, and Cleveland Fed President Loretta Mester, stated that the central bank still needed more confidence to cut interest rates, and that the timing of the move remained uncertain. As a result, investors slightly trimmed their expectations for a September rate cut, and the S&P 500 and Nasdaq 100 indexes fell 0.2% and 0.3%, respectively. Meanwhile, Reddit surged 11% after announcing a partnership with OpenAI, while Take-Two Interactive and DXC Technology fell after issuing disappointing earnings guidance.\nsource: Yahoo\npublished: May 16 2024 20:08\nLocation: US, Wall Street\nDate: September, Thursday\nOrganization: Atlanta Fed, Cleveland Fed, New York Fed, Fed, Reddit, Take-Two Interactive, DXC Technology, OpenAI, Federal Reserve\nTitle: President\nPerson: Loretta Mester, Raphael Bostic, John Williams\nclassification: Business\nsentiment: 0\n</doc>\n"
```

## 链接

我们在这里展示如何将其作为代理的一部分使用。我们使用OpenAI Functions代理，因此我们需要为其设置和安装所需的依赖项。我们还将使用[LangSmith Hub](https://smith.langchain.com/hub)来提取提示，因此我们需要安装它。

```bash
pip install -U langchain-openai langchainhub
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_community.tools.asknews import AskNewsSearch
from langchain_openai import ChatOpenAI
prompt = hub.pull("hwchase17/openai-functions-agent")
llm = ChatOpenAI(temperature=0)
asknews_tool = AskNewsSearch()
tools = [asknews_tool]
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke({"input": "How is the tech sector being affected by fed policy?"})
```

科技行业受联邦政策的影响有哪些？

科技行业受联邦政策的影响有多种方式，特别是涉及人工智能（AI）监管和投资方面。以下是一些与科技行业和联邦政策相关的最新新闻文章：

1. 美国参议院发布了一份两党共同制定的人工智能政策路线图，涉及人工智能使用和发展的共识和分歧领域。该路线图包括知识产权改革、人工智能研究资金、行业特定规定和透明度要求的建议。它还强调了增加人工智能创新资金和国防投资的必要性。[来源：The National Law Review]

2. 由美国参议院多数党领袖查克·舒默（Chuck Schumer）领导的一组跨党派参议员提议，在未来三年内至少拨款320亿美元用于发展人工智能并建立相关保障措施。该提议旨在监管和促进人工智能发展，以维护美国的竞争力并改善生活质量。[来源：Cointelegraph]

3. 美国政府计划限制先进人工智能模型的出口，以防止中国和俄罗斯获取这一技术。此举是为了保护国家安全，防止外国势力滥用人工智能。[来源：O Cafezinho]

4. 美国和中国已讨论了人工智能技术的风险，美国在人工智能军备竞赛中占据领先地位。尽管存在对扼杀创新的担忧，美国提议增加320亿美元的联邦支出用于人工智能，以保持其领先地位。[来源：AOL]

这些文章突出了与人工智能监管、投资和出口限制相关的持续讨论和行动，这些行动正在影响科技行业，以响应联邦政策决定。