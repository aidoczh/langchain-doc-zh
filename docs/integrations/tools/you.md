# You.com 搜索
[you.com API](https://api.you.com) 是一套工具，旨在帮助开发者将LLMs的输出与最新、最准确、最相关的信息联系起来，这些信息可能没有包含在他们的训练数据集中。
## 设置
该工具位于 `langchain-community` 包中。
您还需要设置您的 you.com API 密钥。
```python
%pip install --upgrade --quiet langchain-community
```
```python
import os
os.environ["YDC_API_KEY"] = ""
# 用于链接部分
os.environ["OPENAI_API_KEY"] = ""
## 或者从.env文件中加载YDC_API_KEY
# !pip install --quiet -U python-dotenv
# import dotenv
# dotenv.load_dotenv()
```
设置 [LangSmith](https://smith.langchain.com/) 也是有帮助的，以获得最佳的可观察性。
```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
## 工具使用
```python
from langchain_community.tools.you import YouSearchTool
from langchain_community.utilities.you import YouSearchAPIWrapper
api_wrapper = YouSearchAPIWrapper(num_web_results=1)
tool = YouSearchTool(api_wrapper=api_wrapper)
tool
```
```output
YouSearchTool(api_wrapper=YouSearchAPIWrapper(ydc_api_key='054da371-e73b-47c1-a6d9-3b0cddf0fa3e<__>1Obt7EETU8N2v5f4MxaH0Zhx', num_web_results=1, safesearch=None, country=None, k=None, n_snippets_per_hit=None, endpoint_type='search', n_hits=None))
```
```python
# .invoke 包装了 utility.results
response = tool.invoke("What is the weather in NY")
# .invoke 应该对于每个 `snippet` 都有一个 Document
print(len(response))
for item in response:
    print(item)
```
```output
7
page_content='10 Day Weather-Manhattan, NY\nToday43°/39°1%\nToday\nSun 31 | Day\nGenerally cloudy. High 43F. Winds W at 10 to 15 mph.\n- Humidity54%\n- UV Index0 of 11\n- Sunrise7:19 am\n- Sunset4:38 pm\nSun 31 | Night\nCloudy. Low 39F. Winds light and variable.\n- Humidity70%\n- UV Index0 of 11\n- Moonrise9:13 pmWaning Gibbous\n- Moonset10:28 am\nMon 0145°/33°7%\nMon 01\nMon 01 | Day\nConsiderable cloudiness. High around 45F. Winds light and variable.\n- Humidity71%\n- UV Index1 of 11\n- Sunrise7:19 am\n- Sunset4:39 pm\nMon 01 | Night\nA few clouds. Low 33F. Winds NNW at 5 to 10 mph.\n- Humidity64%\n- UV Index0 of 11\n- Moonrise10:14 pmWaning Gibbous\n- Moonset10:49 am\nTue 0246°/35°4%\nTue 02\nTue 02 | Day\nMainly sunny. High 46F. Winds NW at 5 to 10 mph.\n- Humidity52%\n- UV Index2 of 11\n- Sunrise7:19 am\n- Sunset4:40 pm\nTue 02 | Night\nA few clouds overnight. Low around 35F. Winds W at 5 to 10 mph.\n- Humidity64%\n- UV Index0 of 11\n- Moonrise11:13 pmWaning Gibbous\n- Moonset11:08 am\nWed 0346°/38°4%\nWed 03\nWed 03 | Day' metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': None, 'title': '10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...', 'description': 'Some sun in the morning with increasing clouds during the afternoon. High around 45F. Winds SSE at 5 to 10 mph. ... Cloudy with showers. Low near 40F. Winds SSE at 5 to 10 mph. Chance of rain 60%. ... A steady rain in the morning. Showers continuing in the afternoon.'}
page_content='Radar\nLatest News\nOur Changing World\nYour Privacy\nTo personalize your product experience, we collect data from your device. We also may use or disclose to specific data vendors your precise geolocation data to provide the Services. To learn more please refer to our Privacy Policy.\nChoose how my information is shared' metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': None, 'title': '10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...', 'description': 'Some sun in the morning with increasing clouds during the afternoon. High around 45F. Winds SSE at 5 to 10 mph. ... Cloudy with showers. Low near 40F. Winds SSE at 5 to 10 mph. Chance of rain 60%. ... A steady rain in the morning. Showers continuing in the afternoon.'}
page_content='- Humidity82%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:34 pm\nTue 26 | Night\nCloudy with light rain developing after midnight. Low 47F. Winds light and variable. Chance of rain 80%.\n- Humidity90%\n- UV Index0 of 11\n- Moonrise4:00 pmFull Moon\n- Moonset7:17 am\nWed 2754°/49°93%\nWed 27\nWed 27 | Day\nRain. High 54F. Winds E at 5 to 10 mph. Chance of rain 90%. Rainfall near a half an inch.\n- Humidity93%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:35 pm\nWed 27 | Night\nSteady light rain in the evening. Showers continuing late. Low 49F. Winds light and variable. Chance of rain 70%.\n- Humidity91%\n- UV Index0 of 11\n- Moonrise4:59 pmFull Moon\n- Moonset8:12 am\nThu 2853°/42°19%\nThu 28\nThu 28 | Day\nCloudy skies early will become partly cloudy later in the day. Slight chance of a rain shower. High 53F. Winds WSW at 5 to 10 mph.\n- Humidity77%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:36 pm\nThu 28 | Night\nPartly cloudy skies. Low 42F. Winds W at 5 to 10 mph.\n- Humidity71%\n- UV Index0 of 11' metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': None, 'title': '10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...', 'description': 'Some sun in the morning with increasing clouds during the afternoon. High around 45F. Winds SSE at 5 to 10 mph. ... Cloudy with showers. Low near 40F. Winds SSE at 5 to 10 mph. Chance of rain 60%. ... A steady rain in the morning. Showers continuing in the afternoon.'}
page_content='- Moonrise2:20 amWaning Crescent\n- Moonset12:33 pm\nSun 0740°/29°19%\nSun 07\nSun 07 | Day\nIntervals of clouds and sunshine. High around 40F. Winds NW at 5 to 10 mph.\n- Humidity57%\n- UV Index2 of 11\n- Sunrise7:19 am\n- Sunset4:44 pm\nSun 07 | Night\nA few clouds from time to time. Low 29F. Winds NNW at 5 to 10 mph.\n- Humidity60%\n- UV Index0 of 11\n- Moonrise3:28 amWaning Crescent\n- Moonset1:04 pm\nMon 0840°/32°35%\nMon 08\nMon 08 | Day\nPartly cloudy early followed mostly cloudy skies and a few snow showers later in the day. High near 40F. Winds N at 5 to 10 mph. Chance of snow 40%.\n- UV Index1 of 11\n- Sunrise7:19 am\n- Sunset4:45 pm\nMon 08 | Night\nVariable clouds with snow showers or flurries. Low 32F. Winds NNE at 5 to 10 mph. Chance of snow 60%. Snow accumulations less than one inch.\n- UV Index0 of 11\n- Moonrise4:40 amWaning Crescent\n- Moonset1:43 pm\nLatest News\nOur Changing World\nYour Privacy' metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': None, 'title': '10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...', 'description': 'Some sun in the morning with increasing clouds during the afternoon. High around 45F. Winds SSE at 5 to 10 mph. ... Cloudy with showers. Low near 40F. Winds SSE at 5 to 10 mph. Chance of rain 60%. ... A steady rain in the morning. Showers continuing in the afternoon.'}
page_content='- Humidity91%\n- UV Index0 of 11\n- Moonrise5:50 amWaning Crescent\n- Moonset2:35 pm\nWed 1056°/39°34%\nWed 10\nWed 10 | Day\nA shower or two possible early with partly cloudy skies in the afternoon. Morning high of 56F with temps falling to near 45. Winds SW at 15 to 25 mph. Chance of rain 30%.\n- Humidity66%\n- UV Index1 of 11\n- Sunrise7:19 am\n
## 链接
我们在这里展示如何将其作为[代理](/docs/tutorials/agents)的一部分。我们使用 OpenAI Functions 代理，因此需要设置并安装所需的依赖项。我们还将使用[LangSmith Hub](https://smith.langchain.com/hub)来获取提示，因此需要安装该工具。
```python
# 你需要一个模型来在链中使用
!pip install --upgrade --quiet langchain langchain-openai langchainhub langchain-community
```
```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
instructions = """你是一个助手。"""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
llm = ChatOpenAI(temperature=0)
you_tool = YouSearchTool(api_wrapper=YouSearchAPIWrapper(num_web_results=1))
tools = [you_tool]
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```
```python
agent_executor.invoke({"input": "纽约今天的天气如何？"})
```
# 纽约市今日天气预报
今日纽约市的天气情况如下：
- 体感温度：43°F
![Weather Forecast and Conditions for New York City, NY](https://weather.com/weather/today/l/96f2f84af9a5f5d452eb0574d4e4d8a840c71b05e22264ebdc0056433a642c84)
来源：[The Weather Channel and Weather.com](https://weather.com/weather/today/l/96f2f84af9a5f5d452eb0574d4e4d8a840c71b05e22264ebdc0056433a642c84)
纽约今天的天气如下：
- 体感温度：43°F
- 最高/最低温度：--/39°F
- 风速：3 mph
- 湿度：63%
- 露点温度：31°F
- 气压：30.44 in
- 紫外线指数：0（满分11）
- 能见度：10 英里
- 月相：上弦月
更多详情请访问[天气频道](https://weather.com/weather/today/l/96f2f84af9a5f5d452eb0574d4e4d8a840c71b05e22264ebdc0056433a642c84)。