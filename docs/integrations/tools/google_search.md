# 谷歌搜索

本笔记介绍如何使用谷歌搜索组件。

首先，您需要设置正确的 API 密钥和环境变量。要设置它，请在谷歌云凭据控制台（https://console.cloud.google.com/apis/credentials）中创建 GOOGLE_API_KEY，并使用可编程搜索引擎（https://programmablesearchengine.google.com/controlpanel/create）创建 GOOGLE_CSE_ID。接下来，建议按照[这里](https://stackoverflow.com/questions/37083058/programmatically-searching-google-in-python-using-custom-search)的说明操作。

然后，我们需要设置一些环境变量。

```python
import os
os.environ["GOOGLE_CSE_ID"] = ""
os.environ["GOOGLE_API_KEY"] = ""
```

```python
from langchain_community.utilities import GoogleSearchAPIWrapper
from langchain_core.tools import Tool
search = GoogleSearchAPIWrapper()
tool = Tool(
    name="google_search",
    description="搜索谷歌获取最新结果。",
    func=search.run,
)
```

```python
tool.run("奥巴马的名字是什么？")
```

```output
"夏威夷州。1. 孩子的名字。 （打字或打印）。 2. 性别。 巴拉克。 3. 这个出生。 生存证明。 文件。 编号151 le。 lb。 中间名。 巴拉克·侯赛因·奥巴马二世是一位美国前政治家，曾担任2009年至2017年间的美国第44任总统。他是民主党的成员。当巴拉克·奥巴马在2008年当选总统时，他成为第一位担任这一职务的非裔美国人...中东仍然是一个重要的外交政策挑战。 2017年1月19日...乔丹·巴拉克·特雷杰，2008年出生于纽约市...乔丹·巴拉克·特雷杰成为全国新闻的焦点，当时他是纽约一家报纸的关注对象...乔治·华盛顿的肖像，美国第一任总统...巴拉克·奥巴马的肖像，美国第44任总统...他的全名是巴拉克·侯赛因·奥巴马二世。由于“二世”只是因为他是以父亲的名字命名的，所以他的姓氏是奥巴马。2008年3月22日...巴里·奥巴马决定不喜欢他的绰号。他在奥克西迪纪念学院的一些朋友已经开始称他为巴拉克（他的... 2017年8月18日...他花了几秒钟和多个线索才记起前总统巴拉克·奥巴马的名字。米勒知道每个答案都必须... 2015年2月9日...迈克尔·乔丹在50岁生日礼物上拼错了巴拉克·奥巴马的名字...知道奥巴马是芝加哥人，也是一个狂热的篮球迷... 4天前...巴拉克·奥巴马，全名巴拉克·侯赛因·奥巴马二世，（1961年8月4日出生于美国夏威夷州檀香山），美国第44任总统（2009年至2017年）和..."
```

## 结果数量

您可以使用 `k` 参数设置结果数量。

```python
search = GoogleSearchAPIWrapper(k=1)
tool = Tool(
    name="我在运气好",
    description="搜索谷歌并返回第一个结果。",
    func=search.run,
)
```

```python
tool.run("python")
```

```output
'Python 编程语言的官方主页。'
```

'Python 编程语言的官方主页。'

## 元数据结果

通过 GoogleSearch 运行查询并返回摘要、标题和链接元数据。

- 摘要：结果的描述。

- 标题：结果的标题。

- 链接：结果的链接。

```python
search = GoogleSearchAPIWrapper()
def top5_results(query):
    return search.results(query, 5)
tool = Tool(
    name="Google 搜索摘要",
    description="搜索谷歌获取最新结果。",
    func=top5_results,
)
```