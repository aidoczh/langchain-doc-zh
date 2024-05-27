# Spider

[Spider](https://spider.cloud/) 是最快速且价格最实惠的网络爬虫和数据抓取工具，可返回适用于大型语言模型的数据。

## 设置

```python
pip install spider-client
```

## 使用方法

要使用 Spider，您需要从 [spider.cloud](https://spider.cloud/) 获取一个 API 密钥。

```python
from langchain_community.document_loaders import SpiderLoader
loader = SpiderLoader(
    api_key="YOUR_API_KEY",
    url="https://spider.cloud",
    mode="scrape",  # 如果没有提供 API 密钥，它将查找环境中的 SPIDER_API_KEY
)
data = loader.load()
print(data)
```

```output
[Document(page_content='Spider - Fastest Web Crawler built for AI Agents and Large Language Models[Spider v1 Logo Spider ](/)The World\'s Fastest and Cheapest Crawler API==========View Demo* Basic* StreamingExample requestPythonCopy```import requests, osheaders = {    \'Authorization\': os.environ["SPIDER_API_KEY"],    \'Content-Type\': \'application/json\',}json_data = {"limit":50,"url":"http://www.example.com"}response = requests.post(\'https://api.spider.cloud/crawl\',  headers=headers,  json=json_data)print(response.json())```Example ResponseScrape with no headaches----------* Proxy rotations* Agent headers* Avoid anti-bot detections* Headless chrome* Markdown LLM ResponsesThe Fastest Web Crawler----------* Powered by [spider-rs](https://github.com/spider-rs/spider)* Do 20,000 pages in seconds* Full concurrency* Powerful and simple API* Cost effectiveScrape Anything with AI----------* Custom scripting browser* Custom data extraction* Data pipelines* Detailed insights* Advanced labeling[API](/docs/api) [Price](/credits/new) [Guides](/guides) [About](/about) [Docs](https://docs.rs/spider/latest/spider/) [Privacy](/privacy) [Terms](/eula)© 2024 Spider from A11yWatchTheme Light Dark Toggle Theme [GitHubGithub](https://github.com/spider-rs/spider)', metadata={'description': 'Collect data rapidly from any website. Seamlessly scrape websites and get data tailored for LLM workloads.', 'domain': 'spider.cloud', 'extracted_data': None, 'file_size': 33743, 'keywords': None, 'pathname': '/', 'resource_type': 'html', 'title': 'Spider - Fastest Web Crawler built for AI Agents and Large Language Models', 'url': '48f1bc3c-3fbb-408a-865b-c191a1bb1f48/spider.cloud/index.html', 'user_id': '48f1bc3c-3fbb-408a-865b-c191a1bb1f48'})]
```

## 模式

- `scrape`：默认模式，用于抓取单个 URL

- `crawl`：抓取提供的域名 URL 的所有子页面

## 爬虫选项

`params` 参数是可传递给加载器的字典。查看 [Spider 文档](https://spider.cloud/docs/api) 以查看所有可用参数。