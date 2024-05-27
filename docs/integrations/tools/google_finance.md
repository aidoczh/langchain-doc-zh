# 谷歌财经

本文介绍如何使用谷歌财经工具从谷歌财经页面获取信息。

要获取 SerpApi 密钥，请在以下网址注册：https://serpapi.com/users/sign_up。

然后使用以下命令安装 google-search-results：

```python
pip install google-search-results
```

接下来，将环境变量 SERPAPI_API_KEY 设置为您的 SerpApi 密钥，

或者将密钥作为参数传递给包装器 serp_api_key="your secret key"。

使用该工具

```python
%pip install --upgrade --quiet  google-search-results
```

```python
import os
from langchain_community.tools.google_finance import GoogleFinanceQueryRun
from langchain_community.utilities.google_finance import GoogleFinanceAPIWrapper
os.environ["SERPAPI_API_KEY"] = ""
tool = GoogleFinanceQueryRun(api_wrapper=GoogleFinanceAPIWrapper())
```

```python
tool.run("Google")
```

在 Langchain 中使用

```python
import os
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
os.environ["OPENAI_API_KEY"] = ""
os.environ["SERP_API_KEY"] = ""
llm = OpenAI()
tools = load_tools(["google-scholar", "google-finance"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run("what is google's stock")
```