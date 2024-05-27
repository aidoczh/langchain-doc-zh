# Polygon 股票市场 API 工具
[Polygon](https://polygon.io/) 的 Stocks API 提供了 REST 端点，让您可以查询所有美国股票交易所的最新市场数据。
这个笔记本使用了一些工具来从 Polygon 获取股票市场数据，比如最新报价和新闻。
```python
import getpass
import os
os.environ["POLYGON_API_KEY"] = getpass.getpass()
```
```output
········
```
```python
from langchain_community.tools.polygon.aggregates import PolygonAggregates
from langchain_community.tools.polygon.financials import PolygonFinancials
from langchain_community.tools.polygon.last_quote import PolygonLastQuote
from langchain_community.tools.polygon.ticker_news import PolygonTickerNews
from langchain_community.utilities.polygon import PolygonAPIWrapper
```
```python
api_wrapper = PolygonAPIWrapper()
ticker = "AAPL"
```
### 获取股票的最新报价
```python
# 获取股票的最新报价
last_quote_tool = PolygonLastQuote(api_wrapper=api_wrapper)
last_quote = last_quote_tool.run(ticker)
print(f"工具输出: {last_quote}")
```
```output
工具输出: {"P": 170.5, "S": 2, "T": "AAPL", "X": 11, "i": [604], "p": 170.48, "q": 106666224, "s": 1, "t": 1709945992614283138, "x": 12, "y": 1709945992614268948, "z": 3}
```
```python
import json
# 将最新报价响应转换为 JSON
last_quote = last_quote_tool.run(ticker)
last_quote_json = json.loads(last_quote)
```
```python
# 打印股票的最新价格
latest_price = last_quote_json["p"]
print(f"{ticker} 的最新价格为 ${latest_price}")
```
```output
AAPL 的最新价格为 $170.48
```
### 获取股票的聚合数据（历史价格）
```python
from langchain_community.tools.polygon.aggregates import PolygonAggregatesSchema
# 定义参数
params = PolygonAggregatesSchema(
    ticker=ticker,
    timespan="day",
    timespan_multiplier=1,
    from_date="2024-03-01",
    to_date="2024-03-08",
)
# 获取股票的聚合数据
aggregates_tool = PolygonAggregates(api_wrapper=api_wrapper)
aggregates = aggregates_tool.run(tool_input=params.dict())
aggregates_json = json.loads(aggregates)
```
```python
print(f"总聚合数: {len(aggregates_json)}")
print(f"聚合数据: {aggregates_json}")
```
```output
总聚合数: 6
聚合数据: [{'v': 73450582.0, 'vw': 179.0322, 'o': 179.55, 'c': 179.66, 'h': 180.53, 'l': 177.38, 't': 1709269200000, 'n': 911077}, {'v': 81505451.0, 'vw': 174.8938, 'o': 176.15, 'c': 175.1, 'h': 176.9, 'l': 173.79, 't': 1709528400000, 'n': 1167166}, {'v': 94702355.0, 'vw': 170.3234, 'o': 170.76, 'c': 170.12, 'h': 172.04, 'l': 169.62, 't': 1709614800000, 'n': 1108820}, {'v': 68568907.0, 'vw': 169.5506, 'o': 171.06, 'c': 169.12, 'h': 171.24, 'l': 168.68, 't': 1709701200000, 'n': 896297}, {'v': 71763761.0, 'vw': 169.3619, 'o': 169.15, 'c': 169, 'h': 170.73, 'l': 168.49, 't': 1709787600000, 'n': 825405}, {'v': 76267041.0, 'vw': 171.5322, 'o': 169, 'c': 170.73, 'h': 173.7, 'l': 168.94, 't': 1709874000000, 'n': 925213}]
```
### 获取股票的最新新闻
```python
ticker_news_tool = PolygonTickerNews(api_wrapper=api_wrapper)
ticker_news = ticker_news_tool.run(ticker)
```
```python
# 将新闻响应转换为 JSON 数组
ticker_news_json = json.loads(ticker_news)
print(f"新闻总数: {len(ticker_news_json)}")
```
```output
新闻总数: 10
```
```python
# 查看第一条新闻
news_item = ticker_news_json[0]
print(f"标题: {news_item['title']}")
print(f"描述: {news_item['description']}")
print(f"发布者: {news_item['publisher']['name']}")
print(f"URL: {news_item['article_url']}")
```
```output
标题: An AI surprise could fuel a 20% rally for the S&P 500 in 2024, says UBS
描述: If Gen AI causes a big productivity boost, stocks could see an unexpected rally this year, say UBS strategists.
发布者: MarketWatch
URL: https://www.marketwatch.com/story/an-ai-surprise-could-fuel-a-20-rally-for-the-s-p-500-in-2024-says-ubs-1044d716
```
### 获取股票的财务数据
```python
financials_tool = PolygonFinancials(api_wrapper=api_wrapper)
financials = financials_tool.run(ticker)
```
```python
# 将财务响应转换为 JSON
financials_json = json.loads(financials)
print(f"总报告期数: {len(financials_json)}")
```
```output
总报告期数: 10
```
```python
# 打印最新报告期的财务元数据
financial_data = financials_json[0]
print(f"公司名称: {financial_data['company_name']}")
print(f"CIK: {financial_data['cik']}")
print(f"财政期间: {financial_data['fiscal_period']}")
print(f"结束日期: {financial_data['end_date']}")
print(f"开始日期: {financial_data['start_date']}")
```
```output
公司名称: APPLE INC
CIK: 0000320193
财政期间: TTM
结束日期: 2023-12-30
开始日期: 2022-12-31
```
```python
# 打印最新报告期的利润表
print(f"利润表: {financial_data['financials']['income_statement']}")
```
```output
资产负债表: {'total_assets': {'value': 320400000000.0, 'unit': 'USD', 'label': 'Total Assets', 'order': 100}, 'total_liabilities': {'value': 258750000000.0, 'unit': 'USD', 'label': 'Total Liabilities', 'order': 300}, 'total_equity': {'value': 61650000000.0, 'unit': 'USD', 'label': 'Total Equity', 'order': 200}}
```
资产负债表：
| 项目 | 数值 | 单位 | 排序 |
| ---- | ---- | ---- | ---- |
| 资产 | 353514000000.0 | 美元 | 100 |
| 流动资产 | 143692000000.0 | 美元 | 200 |
| 库存 | 6511000000.0 | 美元 | 230 |
| 其他流动资产 | 137181000000.0 | 美元 | 250 |
| 非流动资产 | 209822000000.0 | 美元 | 300 |
| 固定资产 | 43666000000.0 | 美元 | 320 |
| 其他非流动资产 | 166156000000.0 | 美元 | 350 |
| 负债和权益 | 353514000000.0 | 美元 | 1900 |
| 负债 | 279414000000.0 | 美元 | 600 |
| 流动负债 | 133973000000.0 | 美元 | 700 |
| 应付账款 | 58146000000.0 | 美元 | 710 |
| 其他流动负债 | 75827000000.0 | 美元 | 740 |
| 长期负债 | 145441000000.0 | 美元 | 800 |
| 长期负债 | 106000000000.0 | 美元 | 810 |
| 非控股权益 | 0 | 美元 | 1500 |
| 母公司股东权益 | 74100000000.0 | 美元 | 1600 |
| 权益 | 74100000000.0 | 美元 | 1400 |
现金流量表：
| 项目 | 数值 | 单位 | 排序 |
| ---- | ---- | ---- | ---- |
| 净现金流 | 20000000000.0 | 美元 | 1100 |
| 继续经营的净现金流 | 20000000000.0 | 美元 | 1200 |
| 经营活动产生的净现金流 | 116433000000.0 | 美元 | 100 |
| 继续经营的经营活动产生的净现金流 | 116433000000.0 | 美元 | 200 |
| 投资活动产生的净现金流 | 7077000000.0 | 美元 | 400 |
| 继续投资活动产生的净现金流 | 7077000000.0 | 美元 | 500 |
| 筹资活动产生的净现金流 | -103510000000.0 | 美元 | 700 |
| 继续筹资活动产生的净现金流 | -103510000000.0 | 美元 | 800 |
