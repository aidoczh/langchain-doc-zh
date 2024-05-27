# 搜索工具

这篇笔记展示了各种搜索工具的用法。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
```

## Google Serper API 包装器

首先，让我们尝试使用 Google Serper API 工具。

```python
tools = load_tools(["google-serper"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("What is the weather in Pomfret?")
```

```output
> 进入新的 AgentExecutor 链...
我应该查看当前的天气状况。
动作：搜索
动作输入："weather in Pomfret"
观察：37°F
想法：我现在知道了 Pomfret 的当前温度。
最终答案：Pomfret 当前的温度为 37°F。
> 链结束。
```

```output
'Pomfret 当前的温度为 37°F。'
```

## SearchApi

其次，让我们尝试 SearchApi 工具。

```python
tools = load_tools(["searchapi"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("What is the weather in Pomfret?")
```

```output
> 进入新的 AgentExecutor 链...
我需要找出 Pomfret 的当前天气。
动作：searchapi
动作输入："weather in Pomfret"
观察：周四 14 | 白天 ... 今天早晨有些云，下午晴朗。最高温度 73°F。西北风 5 到 10 英里/小时。
Pomfret, VT 的 10 天天气预报。截至东部夏令时间上午 4:28。今天。68°/48°。4%。周四 14 | 白天。68°。4%。西北西风 10 英里/小时。今天早晨有些云，下午晴朗。最高温度 73°F。西北风 5 到 10 英里/小时。
Pomfret, MD 的最准确的 10 天天气预报。来自 The Weather Channel 和 Weather.com。今天。68°/48°。4%。周四 14 | 白天。68°。4%。西北西风 10 英里/小时。今天早晨有些云，下午晴朗。最高温度 73°F。西北风 5 到 10 英里/小时。
Pomfret CT。过夜。过夜：凌晨 3 点前有零星阵雨，然后凌晨 4 点后有零星阵雾。其他情况，大部分。零星阵雾。最低温度：58°F。周四。
想法：我现在知道了最终答案
最终答案：Pomfret 当前的天气是大部分多云，最高温度约为 67°F，有降雨的可能。风来自北方，18 到 22 英里/小时，阵风高达 34 英里/小时。
> 链结束。
```

```output
'Pomfret 当前的天气是大部分多云，最高温度约为 67°F，有降雨的可能。风来自北方，18 到 22 英里/小时，阵风高达 34 英里/小时。'
```

## SerpAPI

现在，让我们使用 SerpAPI 工具。

```python
tools = load_tools(["serpapi"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("What is the weather in Pomfret?")
```

```output
> 进入新的 AgentExecutor 链...
我需要找出 Pomfret 目前的天气是什么。
动作：搜索
动作输入："weather in Pomfret"
观察：{'type': 'weather_result', 'temperature': '69', 'unit': 'Fahrenheit', 'precipitation': '2%', 'humidity': '90%', 'wind': '1 英里/小时', 'location': 'Pomfret, CT', 'date': '星期日 下午 9:00', 'weather': '晴朗'}
想法：我现在知道了 Pomfret 的当前天气。
最终答案：Pomfret 当前的天气是 69 华氏度，降雨概率为 2%，湿度为 90%，风速为 1 英里/小时。目前是晴朗的。
> 链结束。
```

```output
'Pomfret 当前的天气是 69 华氏度，降雨概率为 2%，湿度为 90%，风速为 1 英里/小时。目前是晴朗的。'
```

## GoogleSearchAPIWrapper

现在，让我们使用官方的 Google Search API 包装器。

```python
tools = load_tools(["google-search"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("What is the weather in Pomfret?")
```

```output
> 进入新的 AgentExecutor 链...
我应该查看当前的天气状况。
动作：Google 搜索
动作输入："weather in Pomfret"
观察：早期有阵雨，后来变成持续的小雨。接近纪录高温。最高温度约为 60°F。西南风 10 到 15 英里/小时。降雨概率 60%。
> 链结束。
```

## SearxNG 元搜索引擎

在这里，我们将使用一个自托管的 SearxNG 元搜索引擎。

```python
tools = load_tools(["searx-search"], searx_host="http://localhost:8888", llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run("Pomfret 的天气如何")
```

> 进入新的 AgentExecutor 链...

我应该查看当前天气

动作：SearX 搜索

动作输入："Pomfret 的天气如何"

观察：上午主要多云，有雪。气温约为 40 华氏度。偏北偏西风，5 到 10 英里/小时。降雪几率为 40%。降雪量不到一英寸。

Pomfret, MD 的 10 天天气预报：今天 49°/ 41°，降雨几率 52%，星期一 27日，白天 49°，降雨几率 52%，东南风 14 英里/小时，多云，偶尔有雨。气温 49 华氏度。风速 10 到 20 英里/小时。降雨几率 50%。

Pomfret, VT 的 10 天天气预报：今天 39°/ 32°，降雨几率 37%，星期三 01日，白天 39°，降雨几率 37%，东北风 4 英里/小时，多云，下午有雪。气温 39 华氏度。

Pomfret, CT 的当前天气：凌晨 1:06，35°F，体感温度 32°；今日天气预报：44°最高，体感温度 50°；今晚天气预报：32°最低。

Pomfret, MD 的天气预报：今日白天 41°，降雨几率 1%，下午 43°，降雨几率 0%，傍晚 35°，降雨几率 3%，过夜 34°，降雨几率 2%。

Pomfret, MD 天气预报 | AccuWeather：当前天气 5:35 PM，35° F，体感温度 36°，阴天，东风 3 英里/小时，阵风 5 英里/小时，空气质量优，更多细节请查看 WinterCast。

Pomfret, VT 天气预报 | AccuWeather：当前天气 11:21 AM，23° F，体感温度 27°，阴天，东南东风 3 英里/小时，阵风 7 英里/小时，空气质量良好，更多细节请查看 WinterCast。

Pomfret Center, CT 天气预报 | AccuWeather：每日当前天气 6:50 PM，39° F，体感温度 36°，空气质量良好，西北风 6 英里/小时，阵风 16 英里/小时，大部分晴朗，更多细节请查看 WinterCast。

12:00 PM · 体感温度 36° · 偏北风 5 英里/小时 · 湿度 43% · 紫外线指数 3/10 · 云量 65% · 降雨量 0 英寸。

Pomfret Center, CT 天气状况 | Weather Underground：热门城市 旧金山，CA 49°F，晴天；纽约曼哈顿，NY 37°F，晴朗；伊利诺伊州席勒公园 (60176) 警告 39°F，大部分多云。

思考：我现在知道最终答案

最终答案：Pomfret 目前的天气主要是多云，上午有雪。气温约为 40 华氏度，偏北偏西风，5 到 10 英里/小时。降雪几率为 40%。

> 链结束。

```output
'Pomfret 目前的天气主要是多云，上午有雪。气温约为 40 华氏度，偏北偏西风，5 到 10 英里/小时。降雪几率为 40%。'
```