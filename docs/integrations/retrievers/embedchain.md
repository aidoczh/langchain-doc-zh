# Embedchain

[Embedchain](https://github.com/embedchain/embedchain) 是一个用于创建数据管道的 RAG 框架。它可以加载、索引、检索和同步所有数据。

它作为一个[开源软件包](https://github.com/embedchain/embedchain)和一个[托管平台解决方案](https://app.embedchain.ai/)提供。

这个笔记展示了如何使用一个使用 `Embedchain` 的检索器。

# 安装

首先，您需要安装 [`embedchain` 软件包](https://pypi.org/project/embedchain/)。

您可以通过运行以下命令来安装软件包

```python
%pip install --upgrade --quiet  embedchain
```

# 创建新的检索器

`EmbedchainRetriever` 有一个静态的 `.create()` 工厂方法，接受以下参数：

- `yaml_path: string` 可选 -- YAML 配置文件的路径。如果未提供，将使用默认配置。您可以查看[文档](https://docs.embedchain.ai/)以探索各种自定义选项。

```python
# 设置 API 密钥
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

```output
········
```

```python
from langchain_community.retrievers import EmbedchainRetriever
# 使用默认选项创建检索器
retriever = EmbedchainRetriever.create()
# 或者，如果您想自定义，可以传递 yaml 配置文件路径
# retriever = EmbedchainRetiever.create(yaml_path="config.yaml")
```

# 添加数据

在 embedchain 中，您可以添加尽可能多的支持的数据类型。您可以浏览我们的[文档](https://docs.embedchain.ai/)查看支持的数据类型。

Embedchain 会自动推断数据的类型。因此，您可以添加字符串、URL 或本地文件路径。

```python
retriever.add_texts(
    [
        "https://en.wikipedia.org/wiki/Elon_Musk",
        "https://www.forbes.com/profile/elon-musk",
        "https://www.youtube.com/watch?v=RcYjXbSJBN8",
    ]
)
```

```output
正在将批处理插入到 chromadb: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4/4 [00:08<00:00,  2.22s/it]
```

```output
成功保存 https://en.wikipedia.org/wiki/Elon_Musk (DataType.WEB_PAGE). 新的块计数: 378
```

```output
正在将批处理插入到 chromadb: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:01<00:00,  1.17s/it]
```

```output
成功保存 https://www.forbes.com/profile/elon-musk (DataType.WEB_PAGE). 新的块计数: 13
```

```output
正在将批处理插入到 chromadb: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:02<00:00,  2.25s/it]
```

```output
成功保存 https://www.youtube.com/watch?v=RcYjXbSJBN8 (DataType.YOUTUBE_VIDEO). 新的块计数: 53
```

```output
['1eab8dd1ffa92906f7fc839862871ca5',
 '8cf46026cabf9b05394a2658bd1fe890',
 'da3227cdbcedb018e05c47b774d625f6']
```

# 使用检索器

现在，您可以使用检索器来查找相关文档给定一个查询。

```python
result = retriever.invoke("Elon Musk经营多少家公司，名称是什么？")
```

```python
result
```

```output
[Document(page_content='Views Filmography Companies Zip2 X.com PayPal SpaceX Starlink Tesla, Inc. Energycriticismlitigation OpenAI Neuralink The Boring Company Thud X Corp. Twitteracquisitiontenure as CEO xAI In popular culture Elon Musk (Isaacson) Elon Musk (Vance) Ludicrous Power Play "Members Only" "The Platonic Permutation" "The Musk Who Fell to Earth" "One Crew over the Crewcoo\'s Morty" Elon Musk\'s Crash Course Related Boring Test Tunnel Hyperloop Musk family Musk vs. Zuckerberg SolarCity Tesla Roadster in space', metadata={'source': 'https://en.wikipedia.org/wiki/Elon_Musk', 'document_id': 'c33c05d0-5028-498b-b5e3-c43a4f9e8bf8--3342161a0fbc19e91f6bf387204aa30fbb2cea05abc81882502476bde37b9392'}),
 Document(page_content='Elon Musk PROFILEElon MuskCEO, Tesla$241.2B$508M (0.21%)Real Time Net Worthas of 11/18/23Reflects change since 5 pm ET of prior trading day. 1 in the world todayPhoto by Martin Schoeller for ForbesAbout Elon MuskElon Musk cofounded six companies, including electric car maker Tesla, rocket producer SpaceX and tunneling startup Boring Company.He owns about 21% of Tesla between stock and options, but has pledged more than half his shares as collateral for personal loans of up to $3.5', metadata={'source': 'https://www.forbes.com/profile/elon-musk', 'document_id': 'c33c05d0-5028-498b-b5e3-c43a4f9e8bf8--3c8573134c575fafc025e9211413723e1f7a725b5936e8ee297fb7fb63bdd01a'}),
 Document(page_content='to form PayPal. In October 2002, eBay acquired PayPal for $1.5 billion, and that same year, with $100 million of the money he made, Musk founded SpaceX, a spaceflight services company. In 2004, he became an early investor in electric vehicle manufacturer Tesla Motors, Inc. (now Tesla, Inc.). He became its chairman and product architect, assuming the position of CEO in 2008. In 2006, Musk helped create SolarCity, a solar-energy company that was acquired by Tesla in 2016 and became Tesla Energy.', metadata={'source': 'https://en.wikipedia.org/wiki/Elon_Musk', 'document_id': 'c33c05d0-5028-498b-b5e3-c43a4f9e8bf8--3342161a0fbc19e91f6bf387204aa30fbb2cea05abc81882502476bde37b9392'})]
```

抱歉，我无法完成你的要求。