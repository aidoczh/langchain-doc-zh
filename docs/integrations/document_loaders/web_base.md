# WebBaseLoader

本文介绍了如何使用 `WebBaseLoader` 从 `HTML` 网页中加载所有文本，并将其转换为我们可以在下游使用的文档格式。如果需要更多定制化的网页加载逻辑，请查看一些子类示例，比如 `IMSDbLoader`, `AZLyricsLoader` 和 `CollegeConfidentialLoader`。

如果你不想担心网站爬取、绕过 JS 阻塞网站以及数据清洗，可以考虑使用 `FireCrawlLoader` 或更快的选项 `SpiderLoader`。

```python
from langchain_community.document_loaders import WebBaseLoader
```

```python
loader = WebBaseLoader("https://www.espn.com/")
```

要绕过 SSL 验证错误，在获取时可以设置 "verify" 选项：

```python
loader.requests_kwargs = {'verify':False}
```

```python
data = loader.load()
```

```python
data
```

```output
[Document(page_content="\n\n\n\n\n\n\n\n\nESPN - Serving Sports Fans. Anytime. Anywhere.\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n
```

```python
"""
# 用这段代码来测试新的自定义 BeautifulSoup 解析器
import requests
from bs4 import BeautifulSoup
html_doc = requests.get("{在此处插入新的网址}")
soup = BeautifulSoup(html_doc.text, 'html.parser')
# Beautiful Soup 逻辑将被导出到 langchain_community.document_loaders.webpage.py
# 例如：transcript = soup.select_one("td[class='scrtext']").text
# BS4 文档可以在这里找到：https://www.crummy.com/software/BeautifulSoup/bs4/doc/
"""
```

## 加载多个网页

您也可以通过将 url 列表传递给加载器来一次加载多个网页。这将按照传入的 url 的顺序返回文档列表。

```python
loader = WebBaseLoader(["https://www.espn.com/", "https://google.com"])
docs = loader.load()
docs
```

```output```

```markdown
# ESPN - 为体育迷提供服务。随时随地。
ESPN - 为体育迷提供服务。随时随地。
        跳转到主要内容
        跳转到导航
MenuESPN
Search
scores
NFLNBANCAAMNCAAWNHLSoccer…MLBNCAAFGolfTennisSports BettingBoxingCFLNCAACricketF1HorseLLWSMMANASCARNBA G LeagueOlympic SportsRacingRN BBRN FBRugbyWNBAWorld Baseball ClassicWWEX GamesXFLMore ESPNFantasyListenWatchESPN+
SUBSCRIBE NOW
NHL: 选择比赛
XFL
MLB: 选择比赛
NCAA 棒球
NCAA 垒球
板球：选择比赛
Mel Kiper 的 NFL 模拟选秀 3.0
快速链接
男子锦标赛挑战
女子锦标赛挑战
NFL 选秀顺序
如何观看 NHL 比赛
梦幻棒球：注册
如何观看 PGA TOUR
收藏夹
      管理收藏夹
自定义 ESPN
注册登录
ESPN 网站
ESPN Deportes
Andscape
espnW
ESPNFC
X Games
SEC Network
ESPN 应用
ESPN
ESPN 梦幻
关注 ESPN
Facebook
Twitter
Instagram
Snapchat
YouTube
The ESPN Daily Podcast
你准备好开幕日了吗？这是你对 MLB 赛季前混乱的指南
等等，Jacob deGrom 现在在游骑兵队？Xander Bogaerts 和 Trea Turner 签约了哪里？Carlos Correa 呢？是的，在开幕日之前你需要了解一下。12hESPN
MLB 赛季间的一切3h2:33
世界大赛赔率、胜场总数、每支球队的赌注
免费玩梦幻棒球！
头条新闻
QB Jackson 请求从乌鸦队交易
消息来源：德州雇佣 Terry 为全职教练
喷气机总经理：不急于签约 Rodgers；Lamar 不是选项
Love 将离开北卡罗来纳，进入转会门户
贝利切克对焦虑的爱国者球迷说：看看过去的 25 年
Embiid 伤缺，哈登将在对阵 Jokic、掘金队的比赛中回归
林奇：Purdy '赢得了首发的权利'
曼联、雷克瑟姆计划在圣地亚哥进行 7 月友谊赛
从表面上看，圣地亚哥骑士超过了道奇队
拉马尔想离开巴尔的摩
马库斯·斯皮尔斯确定最需要拉马尔·杰克逊的两支球队7h2:00
拉马尔会坐在一边吗？乌鸦队会选一个 QB 吗？杰克逊交易请求的内幕3hJamison Hensley
巴拉德、马球队将考虑交易 QB 杰克逊
印第安纳波利斯？华盛顿？Barnwell 为 QB 的交易适合性排名
斯奈德的动荡 24 年
在老板丹·斯奈德的领导下，华盛顿的 NFL 球队在场上和场外都陷入困境
斯奈德在 1999 年购买了 NFL 的一支著名球队。24 年后，随着球队待售，他留下了一系列场上失败和场外丑闻。13hJohn KeimESPN
爱荷华明星再次挺身而出
J-Will：Caitlin Clark 目前是大学体育界最大的品牌8h0:47
“对手越强，她的表现就越好”：Clark 被拿来和 Taurasi 比较
Caitlin Clark 在周日的表现让长期观察者回溯数十年来找到比较。16hKevin Pelton
女子精英八强比分
星期一的比赛
检查你的支架！
NBA 选秀
前景如何走向四强
2023 年 NCAA 锦标赛已经进入四强，ESPN 的 Jonathan Givony 回顾了那些改变了他们 NBA 选秀前景的球员。11hJonathan GivonyAndy Lyons/Getty Images
谈篮球
为什么 AD 需要在场上更加自信9h1:33
为什么 Perk 不会责怪 Kyrie 对小牛队的困境8h1:48
每支球队的现状
新 NFL 力量排名：自由市场后的 1-32 名投票，以及低估的季前动作
自由市场狂潮已经来去。哪些球队改善了他们 2023 年的前景，哪些球队受到了打击？12hNFL Nation 记者Illustration by ESPN
责任在贝利切克身上
Bruschi：批评比尔·贝利切克对爱国者的困境是公平的10h1:27
头条新闻
QB Jackson 请求从乌鸦队交易
消息来源：德州雇佣 Terry 为全职教练
喷气机总经理：不急于签约 Rodgers；Lamar 不是选项
Love 将离开北卡罗来纳，进入转会门户
贝利切克对焦虑的爱国者球迷说：看看过去的 25 年
Embiid 伤缺，哈登将在对阵 Jokic、掘金队的比赛中回归
林奇：Purdy '赢得了首发的权利'
曼联、雷克瑟姆计划在圣地亚哥进行 7 月友谊赛
从表面上看，圣地亚哥骑士超过了道奇队
收藏夹
      管理收藏夹
梦幻主页
自定义 ESPN
注册登录
March Madness Live
ESPNMarch Madness Live
观看每场男子 NCAA 锦标赛比赛直播！ ICYMI1:42
奥斯汀皮的教练、投手和接球手在报复性投球后都被驱逐
奥斯汀皮的投手、接球手和教练在一次投球后都被驱逐，因为早些时候击中了自由人队的 Nathan Keeter，在跑下三垒线时庆祝了一次全垒打。男子锦标赛挑战Illustration by ESPN
男子锦标赛挑战
检查你在 2023 年男子锦标赛挑战中的支架，你可以在整个大舞会期间进行跟踪。女子锦标赛挑战Illustration by ESPN
女子锦标赛挑战
检查你在 2023 年女子锦标赛挑战中的支架，你可以在整个大舞会期间进行跟踪。ESPN+ 梦幻棒球小抄：潜力股、失败者、新秀和终结者
在整个季前赛中你一直读到他们的名字，忘记他们在选秀日会是一件遗憾。ESPN+ 梦幻棒球小抄就是确保这不会发生的一种方式。Steph Chambers/Getty Images
Passan 的 2023 MLB 赛季预览：大胆的预测和更多
开幕日只剩一周了——Jeff Passan 从各个可能的角度为你提供了所有需要了解的内容。Bob Kupbens/Icon Sportswire 摄
2023 年 NFL 自由市场：未签约球员的最佳球队
Ezekiel Elliott 可能会去哪里？让我们把剩下的自由球员和球队匹配起来，找到两个交易候选人的位置。Illustration by ESPN
2023 年 NFL 模拟选秀：Mel Kiper 的第一轮选秀预测
Mel Kiper Jr. 为 NFL 选秀第一轮做出了预测，包括在前五名中预测了一次交易。现在流行的
Anne-Marie Sorvin-USA TODAY S
波士顿棕熊记录追踪器：胜利、得分、里程碑
B 的胜利和得分，以及一些个人的超级级别也是如此。在这里跟踪更新。William Purnell-USA TODAY Sports 强制性信用
2023 年 NFL 完整选秀顺序：所有轮次的 AFC、NFC 球队选择
从卡罗来纳猎豹队的总体第一顺位开始，这里是整个 2023 年 NFL 选秀按轮次划分的情况。如何在 ESPN+ 上观看Gregory Fisher/Icon Sportswire
2023 年 NCAA 男子冰球：结果、支架、如何观看
坦帕的比赛将是惊险刺激的，充满了大量的明星球员、高速进攻和出色的防守。Koji Sasahara/AP 照片/文件
如何观看 PGA 巡回赛、大师赛、PGA 锦标赛和 FedEx 杯季后赛
关于如何在 ESPN 和 ESPN+ 上观看 PGA 巡回赛、大师赛、PGA 锦标赛和 FedEx 杯季后赛的一切你都需要知道。Hailie Lynch/XFL
如何观看 XFL：2023 赛程、球队、球员、新闻等
每场 XFL 比赛都将在 ESPN+ 上直播。找出何时何地你还可以观看这八支球队的比赛。注册参加#1 梦幻棒球游戏
重新激活一个联赛
创建一个联赛
加入一个公共联赛
通过模拟选秀练习
体育博彩
Mike Kropf/AP 照片
疯狂三月的投注 2023：支架赔率、线路、技巧等
2023 年 NCAA 锦标赛支架终于发布了，我们有关于所有三月疯狂比赛的投注所需的一切。注册参加#1 梦幻游戏！
创建一个联赛
加入公共联赛
重新激活
模拟选秀现在
ESPN+
NHL：选择比赛
XFL
MLB：选择比赛
NCAA 棒球
NCAA 垒球
板球：选择比赛
Mel Kiper 的 NFL 模拟选秀 3.0
快速链接
男子锦标赛挑战
女子锦标赛挑战
NFL 选秀顺序
如何观看 NHL 比赛
梦幻棒球：注册
如何观看 PGA TOUR
ESPN 网站
ESPN Deportes
Andscape
espnW
ESPNFC
X Games
SEC Network
ESPN 应用
ESPN
ESPN 梦幻
关注 ESPN
Facebook
Twitter
Instagram
Snapchat
YouTube
The ESPN Daily Podcast
使用 Google 搜索 Images Maps Play YouTube News Gmail Drive 更多 »Web 历史记录 | 设置 | 登录高级搜索广告业务解决方案关于 Google© 2023 - 隐私 - 条款
```

这是一个 Python 代码示例，演示了如何使用 nest_asyncio 库来解决 asyncio 和 jupyter 之间的 bug。首先，通过 pip 安装了 nest_asyncio 库，然后使用 apply() 方法应用了该库。

```python
%pip install --upgrade --quiet  nest_asyncio
# 修复 asyncio 和 jupyter 的 bug
import nest_asyncio
nest_asyncio.apply()
```

安装完成后，输出显示已经满足了 nest_asyncio 库的要求：

```output
Requirement already satisfied: nest_asyncio in /Users/harrisonchase/.pyenv/versions/3.9.1/envs/langchain/lib/python3.9/site-packages (1.5.6)
```

接下来的代码示例展示了如何使用 WebBaseLoader 类加载 ESPN 和 Google 网站的内容，并设置了每秒请求的次数为 1。然后使用 aload() 方法加载了这些网页，并输出了结果。

```python
loader = WebBaseLoader(["https://www.espn.com/", "https://google.com"])
loader.requests_per_second = 1
docs = loader.aload()
docs
```

加载 ESPN 网站的内容如下所示：

[ESPN - Serving Sports Fans. Anytime. Anywhere.](https://www.espn.com/)

加载 Google 网站的内容如下所示：

[Google](https://google.com)

以上内容展示了如何使用 nest_asyncio 库解决 asyncio 和 jupyter 之间的 bug，并演示了如何使用 WebBaseLoader 类加载网页内容。

## 加载 xml 文件，或使用不同的 BeautifulSoup 解析器

您也可以查看 `SitemapLoader` 的示例，了解如何加载站点地图文件，这是使用此功能的一个示例。

```python
loader = WebBaseLoader(
    "https://www.govinfo.gov/content/pkg/CFR-2018-title10-vol3/xml/CFR-2018-title10-vol3-sec431-86.xml"
)
loader.default_parser = "xml"
docs = loader.load()
docs
```

```output
[Document(page_content='\n\n10\nEnergy\n3\n2018-01-01\n2018-01-01\nfalse\nUniform test method for the measurement of energy efficiency of commercial packaged boilers.\nÂ§ 431.86\nSection Â§ 431.86\n\nEnergy\nDEPARTMENT OF ENERGY\nENERGY CONSERVATION\nENERGY EFFICIENCY PROGRAM FOR CERTAIN COMMERCIAL AND INDUSTRIAL EQUIPMENT\nCommercial Packaged Boilers\nTest Procedures\n\n\n\n\n§\u2009431.86\nUniform test method for the measurement of energy efficiency of commercial packaged boilers.\n(a) Scope. This section provides test procedures, pursuant to the Energy Policy and Conservation Act (EPCA), as amended, which must be followed for measuring the combustion efficiency and/or thermal efficiency of a gas- or oil-fired commercial packaged boiler.\n(b) Testing and Calculations. Determine the thermal efficiency or combustion efficiency of commercial packaged boilers by conducting the appropriate test procedure(s) indicated in Table 1 of this section.\n\nTable 1—Test Requirements for Commercial Packaged Boiler Equipment Classes\n\nEquipment category\nSubcategory\nCertified rated inputBtu/h\n\nStandards efficiency metric(§\u2009431.87)\n\nTest procedure(corresponding to\nstandards efficiency\nmetric required\nby §\u2009431.87)\n\n\n\nHot Water\nGas-fired\n≥300,000 and ≤2,500,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\nHot Water\nGas-fired\n>2,500,000\nCombustion Efficiency\nAppendix A, Section 3.\n\n\nHot Water\nOil-fired\n≥300,000 and ≤2,500,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\nHot Water\nOil-fired\n>2,500,000\nCombustion Efficiency\nAppendix A, Section 3.\n\n\nSteam\nGas-fired (all*)\n≥300,000 and ≤2,500,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\nSteam\nGas-fired (all*)\n>2,500,000 and ≤5,000,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\n\u2003\n\n>5,000,000\nThermal Efficiency\nAppendix A, Section 2.OR\nAppendix A, Section 3 with Section 2.4.3.2.\n\n\n\nSteam\nOil-fired\n≥300,000 and ≤2,500,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\nSteam\nOil-fired\n>2,500,000 and ≤5,000,000\nThermal Efficiency\nAppendix A, Section 2.\n\n\n\u2003\n\n>5,000,000\nThermal Efficiency\nAppendix A, Section 2.OR\nAppendix A, Section 3. with Section 2.4.3.2.\n\n\n\n*\u2009Equipment classes for commercial packaged boilers as of July 22, 2009 (74 FR 36355) distinguish between gas-fired natural draft and all other gas-fired (except natural draft).\n\n(c) Field Tests. The field test provisions of appendix A may be used only to test a unit of commercial packaged boiler with rated input greater than 5,000,000 Btu/h.\n[81 FR 89305, Dec. 9, 2016]\n\n\nEnergy Efficiency Standards\n\n', lookup_str='', metadata={'source': 'https://www.govinfo.gov/content/pkg/CFR-2018-title10-vol3/xml/CFR-2018-title10-vol3-sec431-86.xml'}, lookup_index=0)]
```

## 使用代理

有时您可能需要使用代理来绕过 IP 封锁。您可以将代理字典传递给加载器（以及底层的 `requests`）来使用它们。

```python
loader = WebBaseLoader(
    "https://www.walmart.com/search?q=parrots",
    proxies={
        "http": "http://{username}:{password}:@proxy.service.com:6666/",
        "https": "https://{username}:{password}:@proxy.service.com:6666/",
    },
)
docs = loader.load()
```