# Kay.ai

[Kai Data API](https://www.kay.ai/) 是为 RAG 🕵️ 构建的。我们正在筛选世界上最大的数据集，将其作为高质量的嵌入，以便您的 AI 代理可以即时检索上下文。最新模型、快速检索和零基础设施。

这篇笔记向您展示了如何检索 [Kay](https://kay.ai/) 支持的数据集。您目前可以搜索 `SEC Filings` 和 `美国公司的新闻稿`。访问 [kay.ai](https://kay.ai) 获取最新的数据。如有任何问题，请加入我们的 [discord](https://discord.gg/hAnE4e5T6M) 或[在推特上@我们](https://twitter.com/vishalrohra_)

## 安装

首先，安装 [`kay` 包](https://pypi.org/project/kay/)。

```python
!pip install kay
```

您还需要一个 API 密钥：您可以在 [https://kay.ai](https://kay.ai/) 免费获取一个。获取 API 密钥后，您必须将其设置为环境变量 `KAY_API_KEY`。

`KayAiRetriever` 有一个静态的 `.create()` 工厂方法，它接受以下参数：

- `dataset_id: string` 必需 -- 一个 Kay 数据集 id。这是关于特定实体（如公司、人员或地点）的数据集合。例如，尝试 `"company"` 

- `data_type: List[string]` 可选 -- 这是数据集中基于其来源或格式的类别，比如在“company”数据集中的‘SEC Filings’、‘Press Releases’或‘Reports’。例如，尝试 ["10-K", "10-Q", "PressRelease"] 在“company”数据集下。如果留空，Kay 将检索所有类型中最相关的上下文。

- `num_contexts: int` 可选，默认为 6 -- 每次调用 `get_relevant_documents()` 时要检索的文档块数。

## 示例

### 基本的检索器用法

```python
# 设置 API 密钥
from getpass import getpass
KAY_API_KEY = getpass()
```

```output
 ········
```

```python
import os
from langchain_community.retrievers import KayAiRetriever
os.environ["KAY_API_KEY"] = KAY_API_KEY
retriever = KayAiRetriever.create(
    dataset_id="company", data_types=["10-K", "10-Q", "PressRelease"], num_contexts=3
)
docs = retriever.invoke(
    "What were the biggest strategy changes and partnerships made by Roku in 2023??"
)
```

```python
docs
```

```output
[Document(page_content='Company Name: ROKU INC\nCompany Industry: CABLE & OTHER PAY TELEVISION SERVICES\nArticle Title: Roku Is One of Fast Company\'s Most Innovative Companies for 2023\nText: The company launched several new devices, including the Roku Voice Remote Pro; upgraded its most premium player, the Roku Ultra; and expanded its products with a new line of smart home devices such as video doorbells, lights, and plugs integrated into the Roku ecosystem. Recently, the company announced it will launch Roku-branded TVs this spring to offer more choice and innovation to both consumers and Roku TV partners. Throughout 2022, Roku also updated its operating system (OS), the only OS purpose-built for TV, with more personalization features and enhancements across search, audio, and content discovery, launching The Buzz, Sports, and What to Watch, which provides tailored movie and TV recommendations on the Home Screen Menu. The company also released a new feature for streamers, Photo Streams, that allows customers to display and share photo albums through Roku streaming devices. Additionally, Roku unveiled Shoppable Ads, a new ad innovation that makes shopping on TV streaming as easy as it is on social media. Viewers simply press "OK" with their Roku remote on a shoppable ad and proceed to check out with their shipping and payment details pre-populated from Roku Pay, its proprietary payments platform. Walmart was the exclusive retailer for the launch, a first-of-its-kind partnership.', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2022, 2023], 'company_name': 'ROKU INC', 'company_sic_code_description': 'CABLE & OTHER PAY TELEVISION SERVICES', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-03-02T09:30:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku Is One of Fast Company's Most Innovative Companies for 2023"}),
 Document(page_content='Company Name: ROKU INC\nCompany Industry: CABLE & OTHER PAY TELEVISION SERVICES\nArticle Title: Roku Is One of Fast Company\'s Most Innovative Companies for 2023\nText: Finally, Roku grew its content offering with thousands of apps and watching options for users, including content on The Roku Channel, a top five app by reach and engagement on the Roku platform in the U.S. in 2022. In November, Roku released its first feature film, "WEIRD: The Weird Al\' Yankovic Story," a biopic starring Daniel Radcliffe. Throughout the year, The Roku Channel added FAST channels from NBCUniversal and the National Hockey League, as well as an exclusive AMC channel featuring its signature drama "Mad Men." This year, the company announced a deal with Warner Bros. Discovery, launching new channels that will include "Westworld" and "The Bachelor," in addition to 2,000 hours of on-demand content. Read more about Roku\'s journey here . Fast Company\'s Most Innovative Companies issue (March/April 2023) is available online here , as well as in-app via iTunes and on newsstands beginning March 14. About Roku, Inc.\nRoku pioneered streaming to the TV. We connect users to the streaming content they love, enable content publishers to build and monetize large audiences, and provide advertisers with unique capabilities to engage consumers. Roku streaming players and TV-related audio devices are available in the U.S. and in select countries through direct retail sales and licensing arrangements with service operators. Roku TV models are available in the U.S. and select countries through licensing arrangements with TV OEM brands.', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2022, 2023], 'company_name': 'ROKU INC', 'company_sic_code_description': 'CABLE & OTHER PAY TELEVISION SERVICES', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-03-02T09:30:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku Is One of Fast Company's Most Innovative Companies for 2023"}),
 Document(page_content='Company Name: ROKU INC\nCompany Industry: CABLE & OTHER PAY TELEVISION SERVICES\nArticle Title: Roku\'s New NFL Zone Gives Fans Easy Access to NFL Games Right On Time for 2023 Season\nText: In partnership with the NFL, the new NFL Zone offers viewers an easy way to find where to watch NFL live games Today, Roku (NASDAQ: ROKU ) and the National Football League (NFL) announced the recently launched NFL Zone within the Roku Sports experience to kick off the 2023 NFL season. This strategic partnership between Roku and the NFL marks the first official league-branded zone within Roku\'s Sports experience. Available now, the NFL Zone offers football fans a centralized location to find live and upcoming games, so they can spend less time figuring out where to watch the game and more time rooting for their favorite teams. Users can also tune in for weekly game previews, League highlights, and additional NFL content, all within the zone. This press release features multimedia. View the full release here: In partnership with the NFL, Roku\'s new NFL Zone offers viewers an easy way to find where to watch NFL live games (Photo: Business Wire) "Last year we introduced the Sports experience for our highly engaged sports audience, making it simpler for Roku users to watch sports programming," said Gidon Katz, President, Consumer Experience, at Roku. "As we start the biggest sports season of the year, providing easy access to NFL games and content to our millions of users is a top priority for us. We look forward to fans immersing themselves within the NFL Zone and making it their destination to find NFL games.', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2023], 'company_name': 'ROKU INC', 'company_sic_code_description': 'CABLE & OTHER PAY TELEVISION SERVICES', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-09-12T09:00:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku's New NFL Zone Gives Fans Easy Access to NFL Games Right On Time for 2023 Season"})]
```

```python
OPENAI_API_KEY = getpass()
```

```output
········
```

```python
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI
model = ChatOpenAI(model="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```

```python
questions = [
    "What were the biggest strategy changes and partnerships made by Roku in 2023?"
    # "Where is Wex making the most money in 2023?",
]
chat_history = []
for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **Question**: {question} \n")
    print(f"**Answer**: {result['answer']} \n")
```

```output
-> **Question**: What were the biggest strategy changes and partnerships made by Roku in 2023? 
**Answer**: 2023年，Roku与FreeWheel达成战略合作，将Roku领先的广告技术引入FreeWheel客户。此合作旨在推动广告支持的视频点播（AVOD）领域的更大互操作性和自动化。此次合作的重点亮点包括Roku需求应用程序编程接口（dAPI）与FreeWheel电视平台的简化集成，从而实现更好的库存质量控制和改进的发布者产量和收入。此外，发布者现在可以使用Roku平台信号，使广告商能够针对受众进行定位并衡量广告活动绩效，而无需依赖cookie。此合作还涉及使用数据清洁室技术，以实现激活额外数据集，从而更好地为发布者和代理商提供衡量和变现。这些合作和策略旨在支持Roku在AVOD市场的增长。
```