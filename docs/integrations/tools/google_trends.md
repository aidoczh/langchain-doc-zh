# 谷歌趋势

本笔记介绍如何使用谷歌趋势工具获取趋势信息。

首先，您需要在 https://serpapi.com/users/sign_up 注册一个 `SerpApi key`。

然后，您需要使用以下命令安装 `google-search-results`：

```bash
pip install google-search-results
```

接着，您需要将环境变量 `SERPAPI_API_KEY` 设置为您的 `SerpApi key`。

[或者您可以将密钥作为参数传递给包装器 `serp_api_key="your secret key"`]

## 使用该工具

```python
%pip install --upgrade --quiet google-search-results
```

```output
Requirement already satisfied: google-search-results in c:\python311\lib\site-packages (2.4.2)
Requirement already satisfied: requests in c:\python311\lib\site-packages (from google-search-results) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in c:\python311\lib\site-packages (from requests->google-search-results) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in c:\python311\lib\site-packages (from requests->google-search-results) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in c:\python311\lib\site-packages (from requests->google-search-results) (2.1.0)
Requirement already satisfied: certifi>=2017.4.17 in c:\python311\lib\site-packages (from requests->google-search-results) (2023.7.22)
```

```python
import os
from langchain_community.tools.google_trends import GoogleTrendsQueryRun
from langchain_community.utilities.google_trends import GoogleTrendsAPIWrapper
os.environ["SERPAPI_API_KEY"] = ""
tool = GoogleTrendsQueryRun(api_wrapper=GoogleTrendsAPIWrapper())
```

```python
tool.run("Water")
```

```output
'Query: Water\nDate From: Nov 20, 2022\nDate To: Nov 11, 2023\nMin Value: 72\nMax Value: 100\nAverage Value: 84.25490196078431\nPrecent Change: 5.555555555555555%\nTrend values: 72, 72, 74, 77, 86, 80, 82, 88, 79, 79, 85, 82, 81, 84, 83, 77, 80, 85, 82, 80, 88, 84, 82, 84, 83, 85, 92, 92, 100, 92, 100, 96, 94, 95, 94, 98, 96, 84, 86, 84, 85, 83, 83, 76, 81, 85, 78, 77, 81, 75, 76\nRising Related Queries: avatar way of water, avatar the way of water, owala water bottle, air up water bottle, lake mead water level\nTop Related Queries: water park, water bottle, water heater, water filter, water tank, water bill, water world, avatar way of water, avatar the way of water, coconut water, deep water, water cycle, water dispenser, water purifier, water pollution, distilled water, hot water heater, water cooler, sparkling water, american water, micellar water, density of water, tankless water heater, tonic water, water jug'
```