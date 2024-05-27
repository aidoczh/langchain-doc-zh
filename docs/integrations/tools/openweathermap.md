# OpenWeatherMap

本笔记介绍如何使用 `OpenWeatherMap` 组件获取天气信息。

首先，您需要注册一个 `OpenWeatherMap API` 密钥：

1. 前往 OpenWeatherMap 网站并注册一个 API 密钥 [这里](https://openweathermap.org/api/)

2. pip install pyowm

然后，我们需要设置一些环境变量：

1. 将您的 API 密钥保存到 OPENWEATHERMAP_API_KEY 环境变量中

## 使用包装器

```python
import os
from langchain_community.utilities import OpenWeatherMapAPIWrapper
os.environ["OPENWEATHERMAP_API_KEY"] = ""
weather = OpenWeatherMapAPIWrapper()
```

```python
weather_data = weather.run("London,GB")
print(weather_data)
```

```output
在伦敦,英国, 当前天气如下:
详细状态: 多云
风速: 2.57 米/秒, 风向: 240°
湿度: 55%
温度: 
  - 当前: 20.12°C
  - 最高: 21.75°C
  - 最低: 18.68°C
  - 体感温度: 19.62°C
雨量: {}
热指数: 无
云量: 75%
```

## 使用工具

```python
import os
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
os.environ["OPENAI_API_KEY"] = ""
os.environ["OPENWEATHERMAP_API_KEY"] = ""
llm = OpenAI(temperature=0)
tools = load_tools(["openweathermap-api"], llm)
agent_chain = initialize_agent(
    tools=tools, llm=llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent_chain.run("伦敦现在的天气如何？")
```

```output
> 进入新的 AgentExecutor 链...
 我需要了解伦敦现在的天气。
动作: OpenWeatherMap
动作输入: 伦敦,英国
观察结果: 在伦敦,英国, 当前天气如下:
详细状态: 多云
风速: 2.57 米/秒, 风向: 240°
湿度: 56%
温度: 
  - 当前: 20.11°C
  - 最高: 21.75°C
  - 最低: 18.68°C
  - 体感温度: 19.64°C
雨量: {}
热指数: 无
云量: 75%
思考: 我现在知道了伦敦的当前天气。
最终答案: 伦敦当前的天气是多云，风速 2.57 米/秒，风向 240°，湿度 56%，温度 20.11°C，最高温 21.75°C，最低温 18.68°C，热指数为无。
> 链结束。
```

```output
'伦敦当前的天气是多云，风速 2.57 米/秒，风向 240°，湿度 56%，温度 20.11°C，最高温 21.75°C，最低温 18.68°C，热指数为无。'
```