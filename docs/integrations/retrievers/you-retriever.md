# You.com
[you.com API](https://api.you.com) 是一套旨在帮助开发人员将LLMs的输出与最新、最准确、最相关的信息联系起来的工具套件，这些信息可能没有包含在它们的训练数据集中。
## 设置
检索器位于`langchain-community`包中。
您还需要设置您的you.com API密钥。
```python
%pip install --upgrade --quiet langchain-community
```
```python
import os
os.environ["YDC_API_KEY"] = ""
# 用于链接部分
os.environ["OPENAI_API_KEY"] = ""
## 替代方法：从.env文件加载YDC_API_KEY
# !pip install --quiet -U python-dotenv
# import dotenv
# dotenv.load_dotenv()
```
设置[LangSmith](https://smith.langchain.com/)也很有帮助（但不是必需的），以获得最佳的可观察性。
```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_PROJECT"] = 'Experimentz'
```
## 实用程序使用
```python
from langchain_community.utilities import YouSearchAPIWrapper
utility = YouSearchAPIWrapper(num_web_results=1)
utility
```
```python
import json
# .raw_results返回API的未经修改的响应
response = utility.raw_results(query="纽约的天气如何")
# API返回一个带有`hits`键的对象，其中包含一个hits列表
hits = response["hits"]
# 使用`num_web_results=1`，`hits`应该长度为1
print(len(hits))
print(json.dumps(hits, indent=2))
```
```output
1
[
  {
    "description": "Be prepared with the most accurate 10-day forecast for Manhattan, NY with highs, lows, chance of precipitation from The Weather Channel and Weather.com",
    "snippets": [
      "10 Day Weather-Manhattan, NY\nToday43\u00b0/39\u00b01%\nToday\nSun 31 | Day\nGenerally cloudy. High 43F. Winds W at 10 to 15 mph.\n- Humidity54%\n- UV Index0 of 11\n- Sunrise7:19 am\n- Sunset4:38 pm\nSun 31 | Night\nCloudy. Low 39F. Winds light and variable.\n- Humidity70%\n- UV Index0 of 11\n- Moonrise9:13 pmWaning Gibbous\n- Moonset10:28 am\nMon 0145\u00b0/33\u00b07%\nMon 01\nMon 01 | Day\nConsiderable cloudiness. High around 45F. Winds light and variable.\n- Humidity71%\n- UV Index1 of 11\n- Sunrise7:19 am\n- Sunset4:39 pm\nMon 01 | Night\nA few clouds. Low 33F. Winds NNW at 5 to 10 mph.\n- Humidity64%\n- UV Index0 of 11\n- Moonrise10:14 pmWaning Gibbous\n- Moonset10:49 am\nTue 0246\u00b0/35\u00b04%\nTue 02\nTue 02 | Day\nMainly sunny. High 46F. Winds NW at 5 to 10 mph.\n- Humidity52%\n- UV Index2 of 11\n- Sunrise7:19 am\n- Sunset4:40 pm\nTue 02 | Night\nA few clouds overnight. Low around 35F. Winds W at 5 to 10 mph.\n- Humidity64%\n- UV Index0 of 11\n- Moonrise11:13 pmWaning Gibbous\n- Moonset11:08 am\nWed 0346\u00b0/38\u00b04%\nWed 03\nWed 03 | Day",
      "Radar\nLatest News\nOur Changing World\nYour Privacy\nTo personalize your product experience, we collect data from your device. We also may use or disclose to specific data vendors your precise geolocation data to provide the Services. To learn more please refer to our Privacy Policy.\nChoose how my information is shared",
      "- Humidity82%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:34 pm\nTue 26 | Night\nCloudy with light rain developing after midnight. Low 47F. Winds light and variable. Chance of rain 80%.\n- Humidity90%\n- UV Index0 of 11\n- Moonrise4:00 pmFull Moon\n- Moonset7:17 am\nWed 2754\u00b0/49\u00b093%\nWed 27\nWed 27 | Day\nRain. High 54F. Winds E at 5 to 10 mph. Chance of rain 90%. Rainfall near a half an inch.\n- Humidity93%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:35 pm\nWed 27 | Night\nSteady light rain in the evening. Showers continuing late. Low 49F. Winds light and variable. Chance of rain 70%.\n- Humidity91%\n- UV Index0 of 11\n- Moonrise4:59 pmFull Moon\n- Moonset8:12 am\nThu 2853\u00b0/42\u00b019%\nThu 28\nThu 28 | Day\nCloudy skies early will become partly cloudy later in the day. Slight chance of a rain shower. High 53F. Winds WSW at 5 to 10 mph.\n- Humidity77%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:36 pm\nThu 28 | Night\nPartly cloudy skies. Low 42F. Winds W at 5 to 10 mph.\n- Humidity71%\n- UV Index0 of 11",
      "- Moonrise2:20 amWaning Crescent\n- Moonset12:33 pm\nSun 0740\u00b0/29\u00b019%\nSun 07\nSun 07 | Day\nIntervals of clouds and sunshine. High around 40F. Winds NW at 5 to 10 mph.\n- Humidity57%\n- UV Index2 of 11\n- Sunrise7:19 am\n- Sunset4:44 pm\nSun 07 | Night\nA few clouds from time to time. Low 29F. Winds NNW at 5 to 10 mph.\n- Humidity60%\n- UV Index0 of 11\n- Moonrise3:28 amWaning Crescent\n- Moonset1:04 pm\nMon 0840\u00b0/32\u00b035%\nMon 08\nMon 08 | Day\nPartly cloudy early followed mostly cloudy skies and a few snow showers later in the day. High near 40F. Winds N at 5 to 10 mph. Chance of snow 40%.\n- UV Index1 of 11\n- Sunrise7:19 am\n- Sunset4:45 pm\nMon 08 | Night\nVariable clouds with snow showers or flurries. Low 32F. Winds NNE at 5 to 10 mph. Chance of snow 60%. Snow accumulations less than one inch.\n- UV Index0 of 11\n- Moonrise4:40 amWaning Crescent\n- Moonset1:43 pm\nLatest News\nOur Changing World\nYour Privacy",
      "- Humidity91%\n- UV Index0 of 11\n- Moonrise5:50 amWaning Crescent\n- Moonset2:35 pm\nWed 1056\u00b0/39\u00b034%\nWed 10\nWed 10 | Day\nA shower or two possible early with partly cloudy skies in the afternoon. Morning high of 56F with temps falling to near 45. Winds SW at 15 to 25 mph. Chance of rain 30%.\n- Humidity66%\n- UV Index1 of 11\n- Sunrise7:19 am\n- Sunset4:47 pm\nWed 10 | Night\nA few clouds from time to time. Low 39F. Winds WSW at 10 to 20 mph.\n- Humidity64%\n- UV Index0 of 11\n- Moonrise6:56 amWaning Crescent\n- Moonset3:38 pm\nThu 1147\u00b0/38\u00b05%\nThu 11\nThu 11 | Day\nPartly cloudy. High 47F. Winds WSW at 5 to 10 mph.\n- Humidity62%\n- UV Index2 of 11\n- Sunrise7:19 am\n- Sunset4:48 pm\nThu 11 | Night\nMostly clear skies. Low 38F. Winds W at 5 to 10 mph.\n- Humidity66%\n- UV Index0 of 11\n- Moonrise7:52 amNew Moon\n- Moonset4:53 pm\nFri 1248\u00b0/42\u00b019%\nFri 12\nFri 12 | Day\nIntervals of clouds and sunshine. High 48F. Winds WSW at 5 to 10 mph.\n- Humidity62%\n- UV Index2 of 11\n- Sunrise7:18 am\n- Sunset4:49 pm",
      "Sat 1346\u00b0/36\u00b053%\nSat 13\nSat 13 | Day\nCloudy with showers. High 46F. Winds WSW at 10 to 15 mph. Chance of rain 50%.\n- Humidity73%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:50 pm\nSat 13 | Night\nRain showers early transitioning to snow showers late. Low 36F. Winds W at 10 to 15 mph. Chance of precip 50%.\n- Humidity70%\n- UV Index0 of 11\n- Moonrise9:14 amWaxing Crescent\n- Moonset7:33 pm\nSun 1442\u00b0/34\u00b037%\nSun 14\nSun 14 | Day\nSnow showers early will transition to a few showers later. High 42F. Winds WSW at 10 to 15 mph. Chance of rain 40%.\n- Humidity63%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:51 pm\nSun 14 | Night\nVariable clouds with snow showers. Low 34F. Winds W at 10 to 15 mph. Chance of snow 60%. Snow accumulations less than one inch.\n- UV Index0 of 11\n- Moonrise9:44 amWaxing Crescent\n- Moonset8:52 pm\nMon 1540\u00b0/31\u00b051%\nMon 15\nMon 15 | Day",
      "- Humidity70%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:34 pm\nMon 25 | Night\nOvercast with showers at times. Low 43F. Winds light and variable. Chance of rain 40%.\n- Humidity80%\n- UV Index0 of 11\n- Moonrise3:08 pmWaxing Gibbous\n- Moonset6:14 am\nTue 2653\u00b0/45\u00b058%\nTue 26\nTue 26 | Day\nOvercast with rain showers at times. High 53F. Winds E at 5 to 10 mph. Chance of rain 60%.\n- Humidity79%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:34 pm\nTue 26 | Night\nShowers early then scattered thunderstorms developing late. Low near 45F. Winds ESE at 5 to 10 mph. Chance of rain 60%.\n- Humidity93%\n- UV Index0 of 11\n- Moonrise4:00 pmFull Moon\n- Moonset7:17 am\nWed 2751\u00b0/41\u00b058%\nWed 27\nWed 27 | Day\nCloudy with showers. High 51F. Winds WSW at 5 to 10 mph. Chance of rain 60%.\n- Humidity79%\n- UV Index1 of 11\n- Sunrise7:18 am\n- Sunset4:35 pm\nWed 27 | Night\nCloudy with showers. Low 41F. Winds NW at 5 to 10 mph. Chance of rain 60%.\n- Humidity72%\n- UV Index0 of 11\n- Moonrise4:59 pmFull Moon\n- Moonset8:13 am"
    ],
    "thumbnail_url": "https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw",
    "title": "10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...",
    "url": "https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US"
  }
]
```
```python
# .results 返回每个片段的解析结果
response = utility.results(query="纽约的天气如何")
# .results 应该对应每个 `snippet` 返回一个 Document
print(len(response))
print(response)
```
```output
7
```
[文档(page_content='10 天气预报-曼哈顿，纽约\n今天43°/39°1%\n今天\n周日 31 | 白天\n一般多云。高温 43 华氏度。西风，10 到 15 英里/小时。\n- 湿度 54%\n- 紫外线指数 0/11\n- 日出时间上午7:19\n- 日落时间下午4:38\n周日 31 | 夜间\n多云。低温 39 华氏度。风轻微变化。\n- 湿度 70%\n- 紫外线指数 0/11\n- 月升时间晚上9:13上弦月\n- 月落时间上午10:28\n周一 0145°/33°7%\n周一 01\n周一 01 | 白天\n多云。高温约 45 华氏度。风轻微变化。\n- 湿度 71%\n- 紫外线指数 1/11\n- 日出时间上午7:19\n- 日落时间下午4:39\n周一 01 | 夜间\n少云。低温 33 华氏度。偏北西风，5 到 10 英里/小时。\n- 湿度 64%\n- 紫外线指数 0/11\n- 月升时间晚上10:14上弦月\n- 月落时间上午10:49\n周二 0246°/35°4%\n周二 02\n周二 02 | 白天\n晴。高温 46 华氏度。西北风，5 到 10 英里/小时。\n- 湿度 52%\n- 紫外线指数 2/11\n- 日出时间上午7:19\n- 日落时间下午4:40\n周二 02 | 夜间\n夜间少云。低温约 35 华氏度。西风，5 到 10 英里/小时。\n- 湿度 64%\n- 紫外线指数 0/11\n- 月升时间晚上11:13上弦月\n- 月落时间上午11:08\n周三 0346°/38°4%\n周三 03\n周三 03 | 白天', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天气预报-天气频道...', 'description': '准备好了吗？查看曼哈顿，纽约最准确的10天预报，包括最高温度、最低温度、降水概率等信息，来自天气频道和Weather.com'}), 文档(page_content='雷达\n最新消息\n我们变化的世界\n您的隐私\n为了个性化您的产品体验，我们会收集来自您设备的数据。我们还可能使用或披露您的精确地理位置数据给特定数据供应商，以提供服务。要了解更多，请参阅我们的隐私政策。\n选择如何分享我的信息', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天气预报-天气频道...', 'description': '准备好了吗？查看曼哈顿，纽约最准确的10天预报，包括最高温度、最低温度、降水概率等信息，来自天气频道和Weather.com'}), 文档(page_content='- 湿度 82%\n- 紫外线指数 1/11\n- 日出时间上午7:18\n- 日落时间下午4:34\n周二 26 | 夜间\n多云，午夜后有小雨。低温 47 华氏度。风轻微变化。降雨概率 80%。\n- 湿度 90%\n- 紫外线指数 0/11\n- 月升时间下午4:00满月\n- 月落时间上午7:17\n周三 2754°/49°93%\n周三 27\n周三 27 | 白天\n雨。高温 54 华氏度。东风，5 到 10 英里/小时。降雨概率 90%。降雨量约半英寸。\n- 湿度 93%\n- 紫外线指数 1/11\n- 日出时间上午7:18\n- 日落时间下午4:35\n周三 27 | 夜间\n晚间有小雨。淋雨持续到深夜。低温 49 华氏度。风轻微变化。降雨概率 70%。\n- 湿度 91%\n- 紫外线指数 0/11\n- 月升时间下午4:59满月\n- 月落时间上午8:12\n周四 2853°/42°19%\n周四 28\n周四 28 | 白天\n早晨多云，后来天空转为局部多云。有小雨可能。高温 53 华氏度。西南西风，5 到 10 英里/小时。\n- 湿度 77%\n- 紫外线指数 1/11\n- 日出时间上午7:18\n- 日落时间下午4:36\n周四 28 | 夜间\n部分多云。低温 42 华氏度。西风，5 到 10 英里/小时。\n- 湿度 71%\n- 紫外线指数 0/11', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天气预报-天气频道...', 'description': '准备好了吗？查看曼哈顿，纽约最准确的10天预报，包括最高温度、最低温度、降水概率等信息，来自天气频道和Weather.com'}), 文档(page_content='- 月升时间凌晨2:20残月\n- 月落时间下午12:33\n周日 0740°/29°19%\n周日 07\n周日 07 | 白天\n多云间晴。高温约 40 华氏度。西北风，5 到 10 英里/小时。\n- 湿度 57%\n- 紫外线指数 2/11\n- 日出时间上午7:19\n- 日落时间下午4:44\n周日 07 | 夜间\n偶尔多云。低温 29 华氏度。偏北西风，5 到 10 英里/小时。\n- 湿度 60%\n- 紫外线指数 0/11\n- 月升时间凌晨3:28残月\n- 月落时间下午1:04\n周一 0840°/32°35%\n周一 08\n周一 08 | 白天\n早间局部多云，后来天空大部多云，白天有雪。高温约 40 华氏度。偏北风，5 到 10 英里/小时。降雪概率 40%。\n- 紫外线指数 1/11\n- 日出时间上午7:19\n- 日落时间下午4:45\n周一 08 | 夜间\n多云，有雪或零星雪花。低温 32 华氏度。东北偏北风，5 到 10 英里/小时。降雪概率 60%。降雪量不足一英寸。\n- 紫外线指数 0/11\n- 月升时间凌晨4:40残月\n- 月落时间下午1:43\n最新消息\n我们变化的世界\n您的隐私', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天气预报-天气频道...', 'description': '准备好了吗？查看曼哈顿，纽约最准确的10天预报，包括最高温度、最低温度、降水概率等信息，来自天气频道和Weather.com'}), 文档(page_content='- 湿度 91%\n- 紫外线指数 0/11\n- 月升时间上午5:50残月\n- 月落时间下午2:35\n周三 1056°/39°34%\n周三 10\n周三 10 | 白天\n早间可能有阵雨，下午天空局部多云。早晨最高温度 56 华氏度，温度下降至约 45。西南风，15 到 25 英里/小时。降雨概率 30%。\n- 湿度 66%\n- 紫外线指数 1/11\n- 日出时间上午7:19\n- 日落时间下午4:47\n周三 10 | 夜间\n偶尔多云。低温 39 华氏度。西南西风，10 到 20 英里/小时。\n- 湿度 64%\n- 紫外线指数 0/11\n- 月升时间上午6:56残月\n- 月落时间下午3:38\n周四 1147°/38°5%\n周四 11\n周四 11 | 白天\n局部多云。高温 47 华氏度。西南西风，5 到 10 英里/小时。\n- 湿度 62%\n- 紫外线指数 2/11\n- 日出时间上午7:19\n- 日落时间下午4:48\n周四 11 | 夜间\n大部分晴朗。低温 38 华氏度。西风，5 到 10 英里/小时。\n- 湿度 66%\n- 紫外线指数 0/11\n- 月升时间上午7:52新月\n- 月落时间下午4:53\n周五 1248°/42°19%\n周五 12\n周五 12 | 白天\n多云间晴。高温 48 华氏度。西南西风，5 到 10 英里/小时。\n- 湿度 62%\n- 紫外线指数 2/11\n- 日出时间上午7:18\n- 日落时间下午4:49', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天气预报-天气频道...', 'description': '准备好了吗？查看曼哈顿，纽约最准确的10天预报，包括最高温度、最低温度、降水概率等信息，来自天气频道和Weather.com'}), 文档(page_content='周六 1346°/36°53%\n周六 13\n周六 13 | 白天\n多云，有阵雨。高温 46 华氏度。西南西风，10 到 15 英里/小时。降雨概率 50%。\n- 湿度 73%\n- 紫外线指数 1/11\n- 日出时间上午7:18\n- 日落时间下午4:50\n周六 13 | 夜间\n早间有雨，晚间转为雪。低温 36 华氏度。西风，10 到 15 英里/小时。降水概率 50%。\n- 湿度 70%\n- 紫外线指数 0/11\n- 月升时间上午9:14初弦月\n- 月落时间晚上7:33\n周日 1442°/34°37%\n周日 14\n周日 14 | 白天\n早间有阵雪，后来转为雨。高温 42 华氏度。西南西风，10 到 15 英里/小时。降雨概率 40%。\n- 湿度 63%\n- 紫外线指数 1/11\n- 日出时间上午7:18\n- 日落时间下午4:51\n周日 14 | 夜间\n多云，有阵雪。低温 34 华氏度。西风，10 到 15 英里/小时。降雪概率 60%。降雪量不足一英寸。\n- 紫外线指数 0/11\n- 月升时间上午9:44初弦月\n- 月落时间晚上8:52\n周一 1540°/31°51%\n周一 15\n周一 15 | 白天', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm
```python
from langchain_community.retrievers.you import YouRetriever
retriever = YouRetriever(num_web_results=1)
retriever
```
```python
# .invoke wraps utility.results
response = retriever.invoke("纽约的天气如何")
# .invoke should have a Document for each `snippet`
print(len(response))
print(response)
```
```output
7
[Document(page_content='10天天气-曼哈顿，纽约\n今天43°/39°1%\n今天\n周日31 | 白天\n一般多云。最高温度43华氏度。西风10到15英里/小时。\n- 湿度54%\n- 紫外线指数0/11\n- 日出时间上午7:19\n- 日落时间下午4:38\n周日31 | 晚上\n多云。最低温度39华氏度。风轻变。\n- 湿度70%\n- 紫外线指数0/11\n- 月升时间晚上9:13上弦月\n- 月落时间上午10:28\n周一01 45°/33°7%\n周一01\n周一01 | 白天\n多云。最高温度约45华氏度。风轻变。\n- 湿度71%\n- 紫外线指数1/11\n- 日出时间上午7:19\n- 日落时间下午4:39\n周一01 | 晚上\n少云。最低温度33华氏度。北北西风5到10英里/小时。\n- 湿度64%\n- 紫外线指数0/11\n- 月升时间晚上10:14上弦月\n- 月落时间上午10:49\n周二02 46°/35°4%\n周二02\n周二02 | 白天\n阳光明媚。最高温度46华氏度。西北风5到10英里/小时。\n- 湿度52%\n- 紫外线指数2/11\n- 日出时间上午7:19\n- 日落时间下午4:40\n周二02 | 晚上\n过夜有些云。最低温度约35华氏度。西风5到10英里/小时。\n- 湿度64%\n- 紫外线指数0/11\n- 月升时间晚上11:13上弦月\n- 月落时间上午11:08\n周三03 46°/38°4%\n周三03\n周三03 | 白天', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天天气预报-天气频道...', 'description': '准备好最准确的曼哈顿，纽约10天天气预报，包括最高温度、最低温度、降水几率等，来自天气频道和Weather.com的天气频道。'}), Document(page_content='雷达\n最新消息\n我们变化的世界\n您的隐私\n为了个性化您的产品体验，我们会从您的设备收集数据。我们还可能使用或向特定数据供应商披露您的精确地理位置数据以提供服务。要了解更多信息，请参阅我们的隐私政策。\n选择如何共享我的信息', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天天气预报-天气频道...', 'description': '准备好最准确的曼哈顿，纽约10天天气预报，包括最高温度、最低温度、降水几率等，来自天气频道和Weather.com的天气频道。'}), Document(page_content='- 湿度82%\n- 紫外线指数1/11\n- 日出时间上午7:18\n- 日落时间下午4:34\n周二26 | 晚上\n多云，午夜后有轻微降雨。最低温度47华氏度。风轻变。降雨几率80%。\n- 湿度90%\n- 紫外线指数0/11\n- 月升时间下午4:00满月\n- 月落时间上午7:17\n周三27 54°/49°93%\n周三27\n周三27 | 白天\n下雨。最高温度54华氏度。东风5到10英里/小时。降雨几率90%。降雨量接近半英寸。\n- 湿度93%\n- 紫外线指数1/11\n- 日出时间上午7:18\n- 日落时间下午4:35\n周三27 | 晚上\n晚上有轻雨。晚上继续阵雨。最低温度49华氏度。风轻变。降雨几率70%。\n- 湿度91%\n- 紫外线指数0/11\n- 月升时间下午4:59满月\n- 月落时间上午8:12\n周四28 53°/42°19%\n周四28\n周四28 | 白天\n早上多云，后来转为部分多云。有轻微降雨的可能。最高温度53华氏度。西西南风5到10英里/小时。\n- 湿度77%\n- 紫外线指数1/11\n- 日出时间上午7:18\n- 日落时间下午4:36\n周四28 | 晚上\n部分多云。最低温度42华氏度。西风5到10英里/小时。\n- 湿度71%\n- 紫外线指数0/11', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天天气预报-天气频道...', 'description': '准备好最准确的曼哈顿，纽约10天天气预报，包括最高温度、最低温度、降水几率等，来自天气频道和Weather.com的天气频道。'}), Document(page_content='- 月升时间上午2:20下弦月\n- 月落时间下午12:33\n周日07 40°/29°19%\n周日07\n周日07 | 白天\n云和阳光交替出现。最高温度约40华氏度。西北风5到10英里/小时。\n- 湿度57%\n- 紫外线指数2/11\n- 日出时间上午7:19\n- 日落时间下午4:44\n周日07 | 晚上\n时而多云。最低温度29华氏度。北北西风5到10英里/小时。\n- 湿度60%\n- 紫外线指数0/11\n- 月升时间上午3:28下弦月\n- 月落时间下午1:04\n周一08 40°/32°35%\n周一08\n周一08 | 白天\n部分多云，后来多云，白天有几场雪。最高温度约40华氏度。北风5到10英里/小时。降雪几率40%。\n- 紫外线指数1/11\n- 日出时间上午7:19\n- 日落时间下午4:45\n周一08 | 晚上\n多云，有雪或雪花飘落。最低温度32华氏度。东北偏北风5到10英里/小时。降雪几率60%。降雪量小于1英寸。\n- 紫外线指数0/11\n- 月升时间上午4:40下弦月\n- 月落时间下午1:43\n最新消息\n我们变化的世界\n您的隐私', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天天气预报-天气频道...', 'description': '准备好最准确的曼哈顿，纽约10天天气预报，包括最高温度、最低温度、降水几率等，来自天气频道和Weather.com的天气频道。'}), Document(page_content='- 湿度91%\n- 紫外线指数0/11\n- 日出时间上午7:18\n- 日落时间下午4:34\n周一25 | 晚上\n多云时有阵雨。最低温度43华氏度。风轻变。降雨几率40%。\n- 湿度80%\n- 紫外线指数0/11\n- 月升时间下午3:08上弦月\n- 月落时间上午6:14\n周二26 53°/45°58%\n周二26\n周二26 | 白天\n多云，时而有阵雨。最高温度53华氏度。东风5到10英里/小时。降雨几率60%。\n- 湿度79%\n- 紫外线指数1/11\n- 日出时间上午7:18\n- 日落时间下午4:34\n周二26 | 晚上\n早些时候有阵雨，晚上有散发雷雨。最低温度约45华氏度。东南偏东风5到10英里/小时。降雨几率60%。\n- 湿度93%\n- 紫外线指数0/11\n- 月升时间下午4:00满月\n- 月落时间上午7:17\n周三27 51°/41°58%\n周三27\n周三27 | 白天\n多云，有阵雨。最高温度51华氏度。西西南风5到10英里/小时。降雨几率60%。\n- 湿度79%\n- 紫外线指数1/11\n- 日出时间上午7:18\n- 日落时间下午4:35\n周三27 | 晚上\n多云，有阵雨。最低温度41华氏度。西北风5到10英里/小时。降雨几率60%。\n- 湿度72%\n- 紫外线指数0/11\n- 月升时间下午4:59满月\n- 月落时间上午8:13', metadata={'url': 'https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US', 'thumbnail_url': 'https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw', 'title': '曼哈顿，纽约的10天天气预报-天气频道...', 'description': '准备好最准确的曼哈顿，纽约10天天气预报，包括最高温度、最低温度、降水几率等，来自天气频道和Weather.com的天气频道。'})]
```
## 链式调用
```python
# 你需要一个模型来在链式调用中使用
!pip install --upgrade --quiet langchain-openai
```
```python
from langchain_community.retrievers.you import YouRetriever
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
# 设置可运行的对象
runnable = RunnablePassthrough
# 设置检索器，限制来源为一个
retriever = YouRetriever(num_web_results=1)
# 设置模型
model = ChatOpenAI(model="gpt-3.5-turbo-16k")
# 设置输出解析器
output_parser = StrOutputParser()
```
### 调用
```python
# 设置期望回答一个问题的提示
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.
Context: {context}
Question: {question}"""
)
# 设置链式调用
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)
output = chain.invoke({"question": "what is the weather in NY today"})
print(output)
```
```output
纽约市今天的天气是 43°，最高/最低温度为 --/39°。风速为 3 英里/小时，湿度为 63%，空气质量被认为是良好的。
```
### 流式
```python
# 设置期望回答一个问题的提示
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.
Context: {context}
Question: {question}"""
)
# 设置链式调用 - 与上述相同
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)
for s in chain.stream({"question": "what is the weather in NY today"}):
    print(s, end="", flush=True)
```
```output
纽约市今天的天气是 39°F 的最高温和 31°F 的最低温，体感温度为 43°F。风速为 3 英里/小时，湿度为 63%，空气质量被认为是良好的。
```
### 批处理
```python
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)
output = chain.batch(
    [
        {"question": "what is the weather in NY today"},
        {"question": "what is the weather in sf today"},
    ]
)
for o in output:
    print(o)
```
```output
根据提供的上下文，纽约市今天的天气是 43°，最高/最低温度为 --/39°。
根据提供的上下文，旧金山目前是多云，温度为 61°F，湿度为 57%。
```