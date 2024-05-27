# Chaindesk

[Chaindesk平台](https://docs.chaindesk.ai/introduction)可以将来自任何地方的数据（数据源：文本、PDF、Word、PowerPoint、Excel、Notion、Airtable、Google Sheets等）导入到数据存储区（包含多个数据源的容器）中。

然后，您可以通过插件或任何其他大型语言模型（LLM）通过`Chaindesk API`将您的数据存储区连接到ChatGPT。

本文介绍了如何使用[Chaindesk](https://www.chaindesk.ai/)的检索器。

首先，您需要注册Chaindesk账号，创建一个数据存储区，添加一些数据，并获取您的数据存储区API端点URL。您需要[API密钥](https://docs.chaindesk.ai/api-reference/authentication)。

## 查询

现在我们的索引已经设置好了，我们可以设置一个检索器并开始查询。

```python
from langchain_community.retrievers import ChaindeskRetriever
```

```python
retriever = ChaindeskRetriever(
    datastore_url="https://clg1xg2h80000l708dymr0fxc.chaindesk.ai/query",
    # api_key="CHAINDESK_API_KEY", # 可选，如果数据存储区是公开的
    # top_k=10 # 可选
)
```

```python
retriever.invoke("What is Daftpage?")
```

```output
[Document(page_content='✨ Made with DaftpageOpen main menuPricingTemplatesLoginSearchHelpGetting StartedFeaturesAffiliate ProgramGetting StartedDaftpage is a new type of website builder that works like a doc.It makes website building easy, fun and offers tons of powerful features for free. Just type / in your page to get started!Daftpage版权所有 © 2022 Daftpage, Inc.保留所有权利。产品定价模板帮助和支持帮助中心入门特点路线图Twitter联盟计划👾 Discord', metadata={'source': 'https:/daftpage.com/help/getting-started', 'score': 0.8697265}),
 Document(page_content="✨ Made with DaftpageOpen main menuPricingTemplatesLoginSearchHelpGetting StartedFeaturesAffiliate ProgramHelp CenterWelcome to Daftpage’s help center—the one-stop shop for learning everything about building websites with Daftpage.Daftpage is the simplest way to create websites for all purposes in seconds. Without knowing how to code, and for free!Get StartedDaftpage is a new type of website builder that works like a doc.It makes website building easy, fun and offers tons of powerful features for free. Just type / in your page to get started!Start here✨ Create your first site🧱 Add blocks🚀 PublishGuides🔖 Add a custom domainFeatures🔥 Drops🎨 Drawings👻 Ghost mode💀 Skeleton modeCant find the answer you're looking for?mail us at support@daftpage.comJoin the awesome Daftpage community on: 👾 DiscordDaftpage版权所有 © 2022 Daftpage, Inc.保留所有权利。产品定价模板帮助和支持帮助中心入门特点路线图Twitter联盟计划👾 Discord", metadata={'source': 'https:/daftpage.com/help', 'score': 0.86570895}),
 Document(page_content=" is the simplest way to create websites for all purposes in seconds. Without knowing how to code, and for free!Get StartedDaftpage is a new type of website builder that works like a doc.It makes website building easy, fun and offers tons of powerful features for free. Just type / in your page to get started!Start here✨ Create your first site🧱 Add blocks🚀 PublishGuides🔖 Add a custom domainFeatures🔥 Drops🎨 Drawings👻 Ghost mode💀 Skeleton modeCant find the answer you're looking for?mail us at support@daftpage.comJoin the awesome Daftpage community on: 👾 DiscordDaftpage版权所有 © 2022 Daftpage, Inc.保留所有权利。产品定价模板帮助和支持帮助中心入门特点路线图Twitter联盟计划👾 Discord", metadata={'source': 'https:/daftpage.com/help', 'score': 0.8645384})]
```