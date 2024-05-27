# Yahoo Finance 新闻

本文介绍如何使用 `yahoo_finance_news` 工具与一个代理程序。

## 设置

首先，您需要安装 `yfinance` Python 包。

```python
%pip install --upgrade --quiet  yfinance
```

## 使用 Chain 的示例

```python
import os
os.environ["OPENAI_API_KEY"] = "..."
```

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.tools.yahoo_finance_news import YahooFinanceNewsTool
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0.0)
tools = [YahooFinanceNewsTool()]
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

```python
agent_chain.run(
    "What happens today with Microsoft stocks?",
)
```

```output
> 进入新的 AgentExecutor 链...
我应该查看有关微软股票的最新财经新闻。
操作：yahoo_finance_news
操作输入：MSFT
观察结果：Microsoft (MSFT) Gains But Lags Market: What You Should Know
在最新的交易会话中，微软 (MSFT) 收盘价为 $328.79，较前一天上涨了 +0.12%。
想法：我有关于微软股票的最新信息。
最终答案：微软 (MSFT) 收盘价为 $328.79，较前一天上涨了 +0.12%。
> 链结束。
```

```output
'微软 (MSFT) 收盘价为 $328.79，较前一天上涨了 +0.12%。'
```

```python
agent_chain.run(
    "How does Microsoft feels today comparing with Nvidia?",
)
```

```output
> 进入新的 AgentExecutor 链...
我应该比较微软和英伟达今天的情绪。
操作：yahoo_finance_news
操作输入：MSFT
观察结果：Microsoft (MSFT) Gains But Lags Market: What You Should Know
在最新的交易会话中，微软 (MSFT) 收盘价为 $328.79，较前一天上涨了 +0.12%。
想法：我还需要找到英伟达当前的情绪。
操作：yahoo_finance_news
操作输入：NVDA
观察结果： 
想法：我现在知道了微软和英伟达的当前情绪。
最终答案：我无法比较微软和英伟达的情绪，因为我只有关于微软的信息。
> 链结束。
```

```output
'我无法比较微软和英伟达的情绪，因为我只有关于微软的信息。'
```

## YahooFinanceNewsTool 如何工作？

```python
tool = YahooFinanceNewsTool()
```

```python
tool.run("NVDA")
```

```output
'未找到使用 NVDA 代码搜索的公司的新闻。'
```

```python
res = tool.run("AAPL")
print(res)
```

```output
苹果、博通和卡特彼勒的最新研究报告
今日研究日报涵盖了 16 家主要股票的新研究报告，其中包括苹果公司 (AAPL)、博通公司 (AVGO) 和卡特彼勒公司 (CAT)。
苹果股票本月表现最差
根据道琼斯市场数据，苹果 (AAPL) 股票本月表现最差。截至八月份，该股下跌了 4.8%，这使得它成为自 2022 年 12 月以来表现最差的月份。
```