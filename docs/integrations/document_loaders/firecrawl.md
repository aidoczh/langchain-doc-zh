# FireCrawl

[FireCrawl](https://firecrawl.dev/?ref=langchain) 是一个将任何网站爬取并转换为 LLM-ready 数据的工具。它会爬取所有可访问的子页面，并为每个页面提供干净的 markdown 和元数据。无需网站地图。

FireCrawl 处理复杂的任务，如反向代理、缓存、速率限制和被 JavaScript 阻止的内容。由 [mendable.ai](https://mendable.ai) 团队开发。

## 设置

```python
pip install firecrawl-py
```

```output
Requirement already satisfied: firecrawl-py in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (0.0.5)
Requirement already satisfied: requests in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (from firecrawl-py) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (from requests->firecrawl-py) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (from requests->firecrawl-py) (3.6)
Requirement already satisfied: urllib3<3,>=1.21.1 in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (from requests->firecrawl-py) (2.0.7)
Requirement already satisfied: certifi>=2017.4.17 in /Users/nicolascamara/anaconda3/envs/langchain/lib/python3.9/site-packages (from requests->firecrawl-py) (2024.2.2)
Note: you may need to restart the kernel to use updated packages.
```

## 使用

您需要获取自己的 API 密钥。请参阅 https://firecrawl.dev

```python
from langchain_community.document_loaders import FireCrawlLoader
```

```python
loader = FireCrawlLoader(
    api_key="YOUR_API_KEY", url="https://firecrawl.dev", mode="crawl"
)
```

```python
docs = loader.load()
```

```python
docs
```

```output
[Document(page_content='[Skip to content](#skip)\n\n[🔥 FireCrawl](/)\n\n[Playground](/playground)\n[Pricing](/pricing)\n\n[Log In](/signin)\n[Log In](/signin)\n[Sign Up](/signin/signup)\n\n![Slack Logo](/images/slack_logo_icon.png)\n\nNew message in: #coach-gtm\n==========================\n\n@CoachGTM: Your meeting prep for Pied Piper < > WindFlow Dynamics is ready! Meeting starts in 30 minutes\n\nTurn websites into  \n_LLM-ready_ data\n=====================================\n\nCrawl and convert any website into clean markdown\n\nTry now (100 free credits)No credit card required\n\nA product by\n\n[![Mendable Logo](/images/mendable_logo_transparent.png)Mendable](https://mendable.ai)\n\n![Mendable Website Image](/mendable-hero-8.png)\n\nCrawl, Capture, Clean\n---------------------\n\nWe crawl all accessible subpages and give you clean markdown for each. No sitemap required.\n\n    \n      [\\\n        {\\\n          "url": "https://www.mendable.ai/",\\\n          "markdown": "## Welcome to Mendable\\\n            Mendable empowers teams with AI-driven solutions - \\\n            streamlining sales and support."\\\n        },\\\n        {\\\n          "url": "https://www.mendable.ai/features",\\\n          "markdown": "## Features\\\n            Discover how Mendable\'s cutting-edge features can \\\n            transform your business operations."\\\n        },\\\n        {\\\n          "url": "https://www.mendable.ai/pricing",\\\n          "markdown": "## Pricing Plans\\\n            Choose the perfect plan that fits your business needs."\\\n        },\\\n        {\\\n          "url": "https://www.mendable.ai/about",\\\n          "markdown": "## About Us\\\n          \\\n            Learn more about Mendable\'s mission and the \\\n            team behind our innovative platform."\\\n        },\\\n        {\\\n          "url": "https://www.mendable.ai/contact",\\\n          "markdown": "## Contact Us\\\n            Get in touch with us for any queries or support."\\\n        },\\\n        {\\\n          "url": "https://www.mendable.ai/blog",\\\n          "markdown": "## Blog\\\n            Stay updated with the latest news and insights from Mendable."\\\n        }\\\n      ]\n      \n\nNote: The markdown has been edited for display purposes.\n\nWe handle the hard stuff\n------------------------\n\nReverse proxyies, caching, rate limits, js-blocked content and more...\n\n#### Crawling\n\nFireCrawl crawls all accessible subpages, even without a sitemap.\n\n#### Dynamic content\n\nFireCrawl gathers data even if a website uses javascript to render content.\n\n#### To Markdown\n\nFireCrawl returns clean, well formatted markdown - ready for use in LLM applications\n\n#### Continuous updates\n\nSchedule syncs with FireCrawl. No cron jobs or orchestration required.\n\n#### Caching\n\nFireCrawl caches content, so you don\'t have to wait for a full scrape unless new content exists.\n\n#### Built for AI\n\nBuilt by LLM engineers, for LLM engineers. Giving you clean data the way you want it.\n\nPricing Plans\n=============\n\nStarter\n-------\n\n50k credits ($1.00/1k)\n\n$50/month\n\n*   Scrape 50,000 pages\n*   Credits valid for 6 months\n*   2 simultaneous scrapers\\*\n\nSubscribe\n\nStandard\n--------\n\n500k credits ($0.75/1k)\n\n$375/month\n\n*   Scrape 500,000 pages\n*   Credits valid for 6 months\n*   4 simultaneous scrapers\\*\n\nSubscribe\n\nScale\n-----\n\n12.5M credits ($0.30/1k)\n\n$1,250/month\n\n*   Scrape 2,500,000 pages\n*   Credits valid for 6 months\n*   10 simultaneous scrapes\\*\n\nSubscribe\n\n\\* a "scraper" refers to how many scraper jobs you can simultaneously submit.\n\nWhat sites work?\n----------------\n\nFirecrawl is best suited for business websites, docs and help centers.\n\nBuisness websites\n\nGathering business intelligence or connecting company data to your AI\n\nBlogs, Documentation and Help centers\n\nGather content from documentation and other textual sources\n\nSocial Media\n\nComing soon\n\n![Feature 01](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fexample-business-2.b6c6b56a.png&w=1920&q=75)\n\n![Feature 02](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fexample-docs-sites.11eef02d.png&w=1920&q=75)\n\nComing Soon\n-----------\n\n[But I want it now!](https://calendly.com/d/cp3d-rvx-58g/mendable-meeting)\n\\* Schedule a meeting\n\n![Feature 04](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fexample-business-2.b6c6b56a.png&w=1920&q=75)\n\n![Slack Logo](/images/slack_logo_icon.png)\n\nNew message in: #coach-gtm\n==========================\n\n@CoachGTM: Your meeting prep for Pied Piper < > WindFlow Dynamics is ready! Meeting starts in 30 minutes\n\n[🔥](/)\n\nReady to _Build?_\n-----------------\n\n[Meet with us](https://calendly.com/d/cp3d-rvx-58g/mendable-meeting)\n\n[Try 100 queries free](/signin)\n\n[Discord](https://discord.gg/gSmWdAkdwd)\n\nFAQ\n---\n\nFrequently asked questions about FireCrawl\n\nWhat is FireCrawl?\n\nFireCrawl is an advanced web crawling and data conversion tool designed to transform any website into clean, LLM-ready markdown. Ideal for AI developers and data scientists, it automates the collection, cleaning, and formatting of web data, streamlining the preparation process for Large Language Model (LLM) applications.\n\nHow does FireCrawl handle dynamic content on websites?\n\nUnlike traditional web scrapers, FireCrawl is equipped to handle dynamic content rendered with JavaScript. It ensures comprehensive data collection from all accessible subpages, making it a reliable tool for scraping websites that rely heavily on JS for content delivery.\n\nCan FireCrawl crawl websites without a sitemap?\n\nYes, FireCrawl can access and crawl all accessible subpages of a website, even in the absence of a sitemap. This feature enables users to gather data from a wide array of web sources with minimal setup.\n\nWhat formats can FireCrawl convert web data into?\n\nFireCrawl specializes in converting web data into clean, well-formatted markdown. This format is particularly suited for LLM applications, offering a structured yet flexible way to represent web content.\n\nHow does FireCrawl ensure the cleanliness of the data?\n\nFireCrawl employs advanced algorithms to clean and structure the scraped data, removing unnecessary elements and formatting the content into readable markdown. This process ensures that the data is ready for use in LLM applications without further preprocessing.\n\nIs FireCrawl suitable for large-scale data scraping projects?\n\nAbsolutely. FireCrawl offers various pricing plans, including a Scale plan that supports scraping of millions of pages. With features like caching and scheduled syncs, it\'s designed to efficiently handle large-scale data scraping and continuous updates, making it ideal for enterprises and large projects.\n\nWhat measures does FireCrawl take to handle web scraping challenges like rate limits and caching?\n\nFireCrawl is built to navigate common web scraping challenges, including reverse proxies, rate limits, and caching. It smartly manages requests and employs caching techniques to minimize bandwidth usage and avoid triggering anti-scraping mechanisms, ensuring reliable data collection.\n\nHow can I try FireCrawl?\n\nYou can start with FireCrawl by trying our free trial, which includes 100 pages. This trial allows you to experience firsthand how FireCrawl can streamline your data collection and conversion processes. Sign up and begin transforming web content into LLM-ready data today!\n\nWho can benefit from using FireCrawl?\n\nFireCrawl is tailored for LLM engineers, data scientists, AI researchers, and developers looking to harness web data for training machine learning models, market research, content aggregation, and more. It simplifies the data preparation process, allowing professionals to focus on insights and model development.\n\n[🔥](/)\n\n© A product by Mendable.ai - All rights reserved.\n\n[Twitter](https://twitter.com/mendableai)\n[GitHub](https://github.com/sideguide)\n[Discord](https://discord.gg/gSmWdAkdwd)\n\nBacked by![Y Combinator Logo](/images/yc.svg)\n\n![SOC 2 Type II](/soc2type2badge.png)\n\n###### Company\n\n*   [About us](#0)\n    \n*   [Diversity & Inclusion](#0)\n    \n*   [Blog](#0)\n    \n*   [Careers](#0)\n    \n*   [Financial statements](#0)\n    \n\n###### Resources\n\n*   [Community](#0)\n    \n*   [Terms of service](#0)\n    \n*   [Collaboration features](#0)\n    \n\n###### Legals\n\n*   [Refund policy](#0)\n    \n*   [Terms & Conditions](#0)\n    \n*   [Privacy policy](#0)\n    \n*   [Brand Kit](#0)', metadata={'title': 'Home - FireCrawl', 'description': 'FireCrawl crawls and converts any website into clean markdown.', 'language': None, 'sourceURL': 'https://firecrawl.dev/'}),
 Document(page_content='[Skip to content](#skip)\n\n[🔥 FireCrawl](/)\n\n[Playground](/playground)\n[Pricing](/pricing)\n\n[Log In](/signin)\n[Log In](/signin)\n[Sign Up](/signin/signup)\n\nPricing Plans\n=============\n\nStarter\n-------\n\n50k credits ($1.00/1k)\n\n$50/month\n\n*   Scrape 50,000 pages\n*   Credits valid for 6 months\n*   2 simultaneous scrapers\\*\n\nSubscribe\n\nStandard\n--------\n\n500k credits ($0.75/1k)\n\n$375/month\n\n*   Scrape 500,000 pages\n*   Credits valid for 6 months\n*   4 simultaneous scrapers\\*\n\nSubscribe\n\nScale\n-----\n\n12.5M credits ($0.30/1k)\n\n$1,250/month\n\n*   Scrape 2,500,000 pages\n*   Credits valid for 6 months\n*   10 simultaneous scrapes\\*\n\nSubscribe\n\n\\* a "scraper" refers to how many scraper jobs you can simultaneously submit.\n\n[🔥](/)\n\n© A product by Mendable.ai - All rights reserved.\n\n[Twitter](https://twitter.com/mendableai)\n[GitHub](https://github.com/sideguide)\n[Discord](https://discord.gg/gSmWdAkdwd)\n\nBacked by![Y Combinator Logo](/images/yc.svg)\n\n![SOC 2 Type II](/soc2type2badge.png)\n\n###### Company\n\n*   [About us](#0)\n    \n*   [Diversity & Inclusion](#0)\n    \n*   [Blog](#0)\n    \n*   [Careers](#0)\n    \n*   [Financial statements](#0)\n    \n\n###### Resources\n\n*   [Community](#0)\n    \n*   [Terms of service](#0)\n    \n*   [Collaboration features](#0)\n    \n\n###### Legals\n\n*   [Refund policy](#0)\n    \n*   [Terms & Conditions](#0)\n    \n*   [Privacy policy](#0)\n    \n*   [Brand Kit](#0)', metadata={'title': 'FireCrawl', 'description': 'Turn any website into LLM-ready data.', 'language': None, 'sourceURL': 'https://firecrawl.dev/pricing'})]
```

## 模式

- `scrape`: 爬取单个 URL 并返回 Markdown 格式的内容。

- `crawl`: 爬取该 URL 及所有可访问的子页面，并返回每个页面的 Markdown 格式的内容。

```python
loader = FireCrawlLoader(
    api_key="YOUR_API_KEY",
    url="https://firecrawl.dev",
    mode="scrape",
)
```

```python
data = loader.load()
```

```python
data
```

```output```

# 火力爬虫

## 爬虫选项

您还可以向加载程序传递 `params`。这是要传递给爬虫的选项字典。有关更多信息，请参阅[FireCrawl API文档](https://github.com/mendableai/firecrawl-py)。