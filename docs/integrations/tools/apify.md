# Apify

本文介绍如何在 LangChain 中使用 [Apify 集成](/docs/integrations/providers/apify)。

[Apify](https://apify.com) 是一个用于网络爬虫和数据提取的云平台，提供了一个包含一千多个现成应用程序（称为 *Actors*）的[生态系统](https://apify.com/store)，用于各种网络爬虫、抓取和数据提取场景。例如，您可以使用它来提取谷歌搜索结果、Instagram 和 Facebook 的个人资料、亚马逊或 Shopify 的产品、谷歌地图的评论等等。

在本示例中，我们将使用 [Website Content Crawler](https://apify.com/apify/website-content-crawler) Actor，该 Actor 可以深度爬取文档、知识库、帮助中心或博客等网站，并从网页中提取文本内容。然后，我们将这些文档输入到向量索引中，并从中回答问题。

```python
%pip install --upgrade --quiet  apify-client langchain-openai langchain
```

首先，在您的源代码中导入 `ApifyWrapper`：

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.utilities import ApifyWrapper
from langchain_core.documents import Document
```

使用您的 [Apify API 令牌](https://console.apify.com/account/integrations) 初始化它，并且为了本示例的目的，还要使用您的 OpenAI API 密钥：

```python
import os
os.environ["OPENAI_API_KEY"] = "您的 OpenAI API 密钥"
os.environ["APIFY_API_TOKEN"] = "您的 Apify API 令牌"
apify = ApifyWrapper()
```

然后运行 Actor，等待它完成，并从 Apify 数据集中获取其结果到 LangChain 文档加载器中。

请注意，如果您已经在 Apify 数据集中有一些结果，您可以直接使用 `ApifyDatasetLoader` 加载它们，如 [此笔记本](/docs/integrations/document_loaders/apify_dataset) 中所示。在那个笔记本中，您还将找到 `dataset_mapping_function` 的解释，该函数用于将 Apify 数据集记录中的字段映射到 LangChain `Document` 的字段。

```python
loader = apify.call_actor(
    actor_id="apify/website-content-crawler",
    run_input={"startUrls": [{"url": "https://python.langchain.com"}]},
    dataset_mapping_function=lambda item: Document(
        page_content=item["text"] or "", metadata={"source": item["url"]}
    ),
)
```

从爬取的文档中初始化向量索引：

```python
index = VectorstoreIndexCreator().from_loaders([loader])
```

最后，查询向量索引：

```python
query = "LangChain 是什么？"
result = index.query_with_sources(query)
```

```python
print(result["answer"])
print(result["sources"])
```

```output
LangChain 是一种标准接口，通过它您可以与各种大型语言模型（LLMs）进行交互。它提供了可以用于构建语言模型应用程序的模块，还提供了具有记忆能力的链和代理。
https://python.langchain.com/en/latest/modules/models/llms.html, https://python.langchain.com/en/latest/getting_started/getting_started.html
```