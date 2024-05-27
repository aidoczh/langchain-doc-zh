# 异步 Chromium

Chromium 是 Playwright 支持的浏览器之一，Playwright 是一个用于控制浏览器自动化的库。

通过运行 `p.chromium.launch(headless=True)`，我们启动了一个无头 Chromium 实例。

无头模式意味着浏览器在没有图形用户界面的情况下运行。

`AsyncChromiumLoader` 加载页面，然后我们使用 `Html2TextTransformer` 将其转换为文本。

```python
%pip install --upgrade --quiet  playwright beautifulsoup4
!playwright install
```

```python
from langchain_community.document_loaders import AsyncChromiumLoader
urls = ["https://www.wsj.com"]
loader = AsyncChromiumLoader(urls)
docs = loader.load()
docs[0].page_content[0:100]
```

```output
'<!DOCTYPE html><html lang="en"><head><script src="https://s0.2mdn.net/instream/video/client.js" asyn'
```

如果您正在使用 Jupyter 笔记本，可能需要在加载文档之前应用 `nest_asyncio`。

```python
!pip install nest-asyncio
import nest_asyncio
nest_asyncio.apply()
```

```python
from langchain_community.document_transformers import Html2TextTransformer
html2text = Html2TextTransformer()
docs_transformed = html2text.transform_documents(docs)
docs_transformed[0].page_content[0:500]
```

```output
"Skip to Main ContentSkip to SearchSkip to... Select * Top News * What's News *\nFeatured Stories * Retirement * Life & Arts * Hip-Hop * Sports * Video *\nEconomy * Real Estate * Sports * CMO * CIO * CFO * Risk & Compliance *\nLogistics Report * Sustainable Business * Heard on the Street * Barron’s *\nMarketWatch * Mansion Global * Penta * Opinion * Journal Reports * Sponsored\nOffers Explore Our Brands * WSJ * * * * * Barron's * * * * * MarketWatch * * *\n* * IBD # The Wall Street Journal SubscribeSig"
```