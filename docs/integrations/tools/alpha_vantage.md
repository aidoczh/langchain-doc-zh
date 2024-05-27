# 阿尔法贝塔

[阿尔法贝塔](https://www.alphavantage.co) 提供实时和历史金融市场数据，通过一组强大且开发者友好的数据 API 和电子表格。

使用 ``AlphaVantageAPIWrapper`` 来获取货币汇率。

```python
import getpass
import os
os.environ["ALPHAVANTAGE_API_KEY"] = getpass.getpass()
```

```python
from langchain_community.utilities.alpha_vantage import AlphaVantageAPIWrapper
```

```python
alpha_vantage = AlphaVantageAPIWrapper()
alpha_vantage._get_exchange_rate("USD", "JPY")
```

```output
{'Realtime Currency Exchange Rate': {'1. From_Currency Code': 'USD',
  '2. From_Currency Name': 'United States Dollar',
  '3. To_Currency Code': 'JPY',
  '4. To_Currency Name': 'Japanese Yen',
  '5. Exchange Rate': '148.19900000',
  '6. Last Refreshed': '2023-11-30 21:43:02',
  '7. Time Zone': 'UTC',
  '8. Bid Price': '148.19590000',
  '9. Ask Price': '148.20420000'}}
```

`_get_time_series_daily` 方法返回指定全球股票的日期、每日开盘价、每日最高价、每日最低价、每日收盘价和每日成交量，覆盖最新的 100 个数据点。

```python
alpha_vantage._get_time_series_daily("IBM")
```

`_get_time_series_weekly` 方法返回指定全球股票的每周最后交易日、每周开盘价、每周最高价、每周最低价、每周收盘价和每周成交量，涵盖 20 多年的历史数据。

```python
alpha_vantage._get_time_series_weekly("IBM")
```

`_get_quote_endpoint` 方法是时间序列 API 的轻量级替代方案，返回指定符号的最新价格和成交量信息。

```python
alpha_vantage._get_quote_endpoint("IBM")
```

```output
{'Global Quote': {'01. symbol': 'IBM',
  '02. open': '156.9000',
  '03. high': '158.6000',
  '04. low': '156.8900',
  '05. price': '158.5400',
  '06. volume': '6640217',
  '07. latest trading day': '2023-11-30',
  '08. previous close': '156.4100',
  '09. change': '2.1300',
  '10. change percent': '1.3618%'}}
```

`search_symbol` 方法根据输入的文本返回符号列表和匹配的公司信息。

```python
alpha_vantage.search_symbols("IB")
```

`_get_market_news_sentiment` 方法返回给定资产的实时和历史市场新闻情绪。

```python
alpha_vantage._get_market_news_sentiment("IBM")
```

`_get_top_gainers_losers` 方法返回美国市场前 20 名涨跌幅最大和活跃度最高的股票。

```python
alpha_vantage._get_top_gainers_losers()
```

包装器的 `run` 方法接受以下参数：from_currency, to_currency。获取给定货币对的货币汇率。

```python
alpha_vantage.run("USD", "JPY")
```

```output
{'1. From_Currency Code': 'USD',
 '2. From_Currency Name': 'United States Dollar',
 '3. To_Currency Code': 'JPY',
 '4. To_Currency Name': 'Japanese Yen',
 '5. Exchange Rate': '148.19900000',
 '6. Last Refreshed': '2023-11-30 21:43:02',
 '7. Time Zone': 'UTC',
 '8. Bid Price': '148.19590000',
 '9. Ask Price': '148.20420000'}
```