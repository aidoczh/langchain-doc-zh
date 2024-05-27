# AskNews

[AskNews](https://asknews.app) 通过单一的自然语言查询为任何 LLM 注入最新的全球新闻（或历史新闻）。具体来说，AskNews 通过翻译、总结、提取实体并将它们索引到热向量和冷向量数据库中，每天丰富超过 30 万篇文章。AskNews 将这些向量数据库放在低延迟的端点上供您使用。当您查询 AskNews 时，您将得到一个经过优化的提示字符串，其中包含所有最相关的丰富信息（例如实体、分类、翻译、总结）。这意味着您无需管理自己的新闻 RAG，也无需担心如何以简洁的方式传达新闻信息给您的 LLM。

AskNews 还致力于透明度，这就是为什么我们的报道受到监控，并在数百个国家、13 种语言和 5 万个来源之间进行了多样化。如果您想追踪我们的来源覆盖范围，可以访问我们的[透明度仪表板](https://asknews.app/en/transparency)。

## 设置

集成位于 `langchain-community` 包中。我们还需要安装 `asknews` 包本身。

```bash
pip install -U langchain-community asknews
```

我们还需要设置我们的 AskNews API 凭据，这可以在[AskNews 控制台](https://my.asknews.app)上生成。

```python
import getpass
import os
os.environ["ASKNEWS_CLIENT_ID"] = getpass.getpass()
os.environ["ASKNEWS_CLIENT_SECRET"] = getpass.getpass()
```

设置 [LangSmith](https://smith.langchain.com/) 也很有帮助（但不是必需的），以获得最佳的可观察性。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

```python
from langchain_community.retrievers import AskNewsRetriever
retriever = AskNewsRetriever(k=3)
retriever.invoke("impact of fed policy on the tech sector")
```

```output
[Document(page_content='[1]:\ntitle: US Stock Market Declines Amid High Interest Rates\nsummary: The US stock market has experienced a significant decline in recent days, with the S&P 500 index falling 9.94 points or 0.2% to 4320.06. The decline was attributed to interest rates, which are expected to remain high for a longer period. The yield on 10-year Treasury notes rose to 4.44%, the highest level since 2007, which has a negative impact on stock prices. The high interest rates have also affected the technology sector, with companies such as Intel and Microsoft experiencing declines. The auto sector is also experiencing fluctuations, with General Motors and Ford experiencing declines. The labor market is also facing challenges, with workers demanding higher wages and benefits, which could lead to increased inflation. The Federal Reserve is expected to keep interest rates high for a longer period, which could have a negative impact on the stock market. Some economists expect the Fed to raise interest rates again this year, which could lead to further declines in the stock market.\nsource: ALYAUM Holding Group for Press\npublished: May 12 2024 13:12\nLocation: US\nTechnology: S&P 500\nQuantity: 9.94 points\nNumber: 4320.06\nProduct: 10-year Treasury notes\nDate: 2007, this year\nOrganization: General Motors, Fed, Intel, Ford, Microsoft, Federal Reserve\nclassification: Finance\nsentiment: -1', metadata={'title': 'الأسهم الأمريكية تتطلع لتعويض خسائرها بعد موجة تراجع كبيرة', 'source': 'https://www.alyaum.com/articles/6529353/%D8%A7%D9%84%D8%A7%D9%82%D8%AA%D8%B5%D8%A7%D8%AF/%D8%A3%D8%B3%D9%88%D8%A7%D9%82-%D8%A7%D9%84%D8%A3%D8%B3%D9%87%D9%85/%D8%A7%D9%84%D8%A3%D8%B3%D9%87%D9%85-%D8%A7%D9%84%D8%A3%D9%85%D8%B1%D9%8A%D9%83%D9%8A%D8%A9-%D8%AA%D8%AA%D8%B7%D9%84%D8%B9-%D9%84%D8%AA%D8%B9%D9%88%D9%8A%D8%B6-%D8%AE%D8%B3%D8%A7%D8%A6%D8%B1%D9%87%D8%A7-%D8%A8%D8%B9%D8%AF-%D9%85%D9%88%D8%AC%D8%A9-%D8%AA%D8%B1%D8%A7%D8%AC%D8%B9-%D9%83%D8%A8%D9%8A%D8%B1%D8%A9', 'images': 'https://www.alyaum.com/uploads/images/2024/05/12/2312237.jpg'}),
 Document(page_content="[2]:\ntitle: US Federal Reserve's Decision to Limit Stock Market Correction\nsummary: The Federal Reserve of the United States, led by Jerome Powell, has achieved its goal of limiting the correction of the stock market by reducing the balance of the central bank and maintaining massive liquidity injections into the markets to combat various crises that have occurred since the pandemic. Despite April's contraction of around 5%, the stock market has behaved well this week, with most indices showing increases. The Dow Jones and S&P 500 have risen almost 2% after declines of around 5% in April, while the Nasdaq has increased by 1.4% after a decline of over 4% in April. The correction is taking place in an orderly manner, and the market is trying to find a new equilibrium in asset valuations, adapted to a normalized cost of money and a moderate but positive growth framework.\nsource: okdiario.com\npublished: May 12 2024 04:45\nOrganization: Federal Reserve of the United States, Dow Jones, S&P 500, Nasdaq\nPerson: Jerome Powell\nEvent: pandemic\nDate: April\nclassification: Business\nsentiment: 1", metadata={'title': 'Las Bolsas afrontan una corrección ordenada apoyas por la Fed de EEUU', 'source': 'https://okdiario.com/economia/reserva-federal-mantiene-liquidez-asegura-correccion-limitada-bolsas-12798172', 'images': 'https://okdiario.com/img/2023/08/25/bild-powell-subida-de-tipos-interior.jpg'}),
 Document(page_content="[3]:\ntitle: How the Fed's quest for transparency made markets more volatile\nsummary: The Federal Reserve's increased transparency and communication with the public may be contributing to market volatility, according to some experts. The Fed's forecasting strategy and frequent communication may be causing \nsource: NBC4 Washington\npublished: May 11 2024 12:00\nOrganization: Fed, Federal Reserve\nclassification: Business\nsentiment: 0", metadata={'title': "How the Fed's quest for transparency made markets more volatile", 'source': 'https://www.nbcwashington.com/news/business/money-report/how-the-feds-quest-for-transparency-made-markets-more-volatile/3613897', 'images': 'https://media.nbcwashington.com/2024/05/107409380-1714652843711-gettyimages-2151006318-_s2_5018_hwe7dfbl.jpeg?quality=85&strip=all&resize=1200%2C675'})]
```

```python
# 你可以完全控制按类别、时间、分页甚至搜索方法进行过滤。
from datetime import datetime, timedelta
start = (datetime.now() - timedelta(days=7)).timestamp()
end = datetime.now().timestamp()
retriever = AskNewsRetriever(
    k=3,
    categories=["Business", "Technology"],
    start_timestamp=int(start),  # 默认为 48 小时前
    end_timestamp=int(end),  # 默认为现在
    method="kw",  # 默认为 "nl"，自然语言，也可以选择 "kw" 进行关键词搜索
    offset=10,  # 允许你分页查看结果
)
retriever.invoke("federal reserve S&P500")
```

```output
[Document(page_content="[1]:\ntitle: 美股在通胀数据公布前以微弱涨势结束本周\nsummary: 美股以微弱涨势结束了本周，标志着三大指数再次实现周涨，投资者们正在评估美联储官员的讲话，并等待下周关键的通胀数据。标普500指数和道琼斯指数略有增长，而纳斯达克指数几乎没有变化。道琼斯指数录得自去年12月中旬以来的最大周涨幅。美联储官员的讲话引发了市场预期，市场参与者正在等待下周公布的通胀数据。数据包括劳工部公布的消费者物价指数（CPI）和生产者物价指数（PPI），预计将为达到2%通胀目标提供见解。\nsource: El Cronista\npublished: May 10 2024 23:35\nLocation: US\nOrganization: 道琼斯, 劳工部, 纳斯达克, 美联储\nDate: 下周, 12 月中旬\nclassification: Business\nsentiment: 0", metadata={'title': 'Modesta suba en los mercados a la espera de los datos de inflación', 'source': 'http://www.cronista.com/usa/wall-street-dolar/modesta-suba-en-los-mercados-a-la-espera-de-los-datos-de-inflacion', 'images': 'https://www.cronista.com/files/image/141/141554/5ff7985549d06_600_315!.jpg?s=99126c63cc44ed5c15ed2177cb022f55&d=1712540173'}),
 Document(page_content="[2]:\ntitle: 美股以强劲姿态结束本周\nsummary: 美国股市以积极态势结束了本周，道琼斯工业平均指数收盘上涨 0.32%，报 39,512.84 点，标普500指数上涨 0.16%，报 5,222.68 点，纳斯达克100指数上涨 0.26%，报 18,161.18 点。三大指数均录得本周涨幅，其中道琼斯指数以 2.2% 的涨幅领先。美联储对利率的立场是一个关键因素，几位美联储成员对未来近期降息表示谨慎。密歇根大学的消费者信心调查显示 5 月份下降，消费者通胀预期也增加，这抑制了降息的预期。尽管如此，市场仍然表现坚挺，并以强劲姿态结束了本周。\nsource: Investing.com\npublished: May 10 2024 20:19\nLocation: US\nQuantity: 39,512.84\nOrganization: 密歇根大学, 美联储, 美联储\nDate: May\nclassification: Business\nsentiment: 1", metadata={'title': 'Aktien New York Schluss: Erneut höher zum Ende einer starken Börsenwoche', 'source': 'https://de.investing.com/news/economy/aktien-new-york-schluss-erneut-hoher-zum-ende-einer-starken-borsenwoche-2618612', 'images': 'https://i-invdn-com.investing.com/news/LYNXMPED2T082_L.jpg'}),
 Document(page_content="[3]:\ntitle: 美股尽管通胀担忧微幅上涨结束本周\nsummary: 美国股市尽管存在通胀预期以及美联储成员对可能降息的怀疑，但本周以微幅上涨结束。道琼斯工业平均指数上涨 0.24%，报 39,480.38 点，标普500指数上涨 0.07%，报 5,217.67 点，纳斯达克100指数也上涨 0.15%，报 18,140.02 点。本周的表现受多种因素推动，包括通胀预期的发布以及美联储成员的讲话。密歇根大学的消费者信心调查也显示 5 月份下降，消费者通胀预期上升。道指表现最佳的 3M 股票在汇丰银行的分析师将其评级升级后上涨 2%。\nsource: Yahoo\npublished: May 10 2024 18:11\nLocation: US\nOrganization: 密歇根大学, 3M, 汇丰银行, 美联储\nQuantity: 39,480.38, 5,217.67\nDate: May\nclassification: Business\nsentiment: 0", metadata={'title': 'Aktien New York: Knapp im Plus gegen Ende einer starken Börsenwoche', 'source': 'https://de.finance.yahoo.com/nachrichten/aktien-new-york-knapp-plus-181143398.html', 'images': 'https://s.yimg.com/cv/apiv2/social/images/yahoo_default_logo-1200x1200.png'})]
```

## 链接

我们可以轻松地将这个检索器链接到一个链中。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
```

```python
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_template(
    """以下新闻文章可能对回答问题有所帮助：
{context}
问题：
{question}"""
)
chain = (
    RunnablePassthrough.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | ChatOpenAI(model="gpt-4-1106-preview")
    | StrOutputParser()
)
chain.invoke({"question": "联邦政策对科技行业的影响是什么？"})
```

```output
"根据第二篇新闻文章提供的信息，美联储政策对科技行业的影响是负面的。文章提到，由于预期利率将在较长时期内保持较高，美国股市出现了显著下跌。这种利率上升特别影响了科技行业，像英特尔和微软等公司都经历了下滑。高利率会导致企业借款成本增加，从而抑制投资和支出。在科技行业，企业经常依靠借款来资助研发和其他增长计划，更高的利率尤其具有挑战性。
因此，美联储保持高利率政策对科技股产生了不利影响，导致它们的市场估值下降。"
```