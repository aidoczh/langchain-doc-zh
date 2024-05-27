# DuckDuckGo 搜索

本文介绍了如何使用 DuckDuckGo 搜索组件。

```python
%pip install --upgrade --quiet  duckduckgo-search
```
```python
from langchain_community.tools import DuckDuckGoSearchRun
```
```python
search = DuckDuckGoSearchRun()
```
```python
search.run("奥巴马的名字是什么？")
```
```output
"奥巴马总统任期结束后的生活 奥巴马身高是多少？ 奥巴马的书籍和格莱美奖 爱好 奥巴马的电影 奥巴马语录 1961 年至今 谁是巴拉克·奥巴马？巴拉克·奥巴马是美国第 44 任总统……你可能不知道的巴拉克·奥巴马的事实之一是，他的直系家人分布在三大洲。奥巴马来自一个有七个活着的同父异母兄弟姐妹的大家庭。他的父亲巴拉克·奥巴马·西尼尔在 1960 年遇见了他的母亲安·邓恩，并在一年后与她结婚。巴拉克总统在白宫东厅发表讲话时，眼泪夺眶而出，回忆起 2012 年桑迪胡克小学 20 名一年级学生的遇难，前第一夫人罗莎琳·卡特于 11 月 29 日在她的家乡乔治亚州普莱恩斯的家中安葬，之前在她的家乡举行了为期三天的悼念仪式。她于 11 月 19 日去世，享年 96 岁……以下是奥巴马总统任期内的 28 项最重要成就。1 - 从大萧条中拯救国家，将失业率从 10% 降至 4.7%……"
```

要获取更多额外信息（例如链接、来源），请使用 `DuckDuckGoSearchResults()`。

```python
from langchain_community.tools import DuckDuckGoSearchResults
```
```python
search = DuckDuckGoSearchResults()
```
```python
search.run("奥巴马")
```
```output
'[snippet: 1:12. 前总统巴拉克·奥巴马在 CNN 的一次采访中表示，他不认为乔·拜登总统在 2024 年的连任中会面临严重的初选挑战……, title: 巴拉克·奥巴马在拜登的 CNN 采访中的五大看点…… - 雅虎, link: https://www.usatoday.com/story/news/politics/2023/06/23/five-takeaways-from-barack-obama-cnn-interview/70349112007/], [snippet: 美国和世界各地的民主机构“变得不堪一击”，前总统巴拉克·奥巴马在一次独家 CNN 采访中警告说，现任总统特朗普……, title: 奥巴马警告民主机构“变得不堪一击”，但特朗普…… - CNN, link: https://www.cnn.com/2023/06/22/politics/barack-obama-interview-cnntv/index.html], [snippet: 巴拉克·奥巴马是美国第 44 任总统，也是第一位黑人总司令。他连任两届，从 2009 年到 2017 年。作为来自肯尼亚和堪萨斯的父母的儿子，奥巴马……, title: 巴拉克·奥巴马：传记，第 44 任美国总统，政治家, link: https://www.biography.com/political-figures/barack-obama], [snippet: 2023 年 8 月 2 日，太平洋夏令时间下午 5:00。迈克·梅莫利和克里斯汀·韦尔克。华盛顿 —— 在 6 月份访问白宫期间，前总统巴拉克·奥巴马向他的前竞选搭档明确表示……, title: 奥巴马私下告诉拜登，他将尽一切努力帮助 2024 年, link: https://www.nbcnews.com/politics/white-house/obama-privately-told-biden-whatever-takes-help-2024-rcna97865], [snippet: 奥巴马基金会副总裁娜塔莉·布基-贝克（Natalie Bookey-Baker）曾在白宫为时任第一夫人米歇尔·奥巴马工作，她表示预计将有约 2500 名校友参加。他们是奥巴马……, title: 本周芝加哥举行巴拉克·奥巴马团队重聚活动；预计将有 2500 名校友参加……, link: https://chicago.suntimes.com/politics/2023/10/29/23937504/barack-obama-michelle-obama-david-axelrod-pod-save-america-jon-batiste-jen-psaki-reunion-obamaworld]'
```

您还可以搜索新闻文章。使用关键字 ``backend="news"``。

```python
search = DuckDuckGoSearchResults(backend="news")
```
```python
search.run("奥巴马")
```
```output
'[snippet: 1:12. 前总统巴拉克·奥巴马在 CNN 的一次采访中表示，他不认为乔·拜登总统在 2024 年的连任中会面临严重的初选挑战……, title: 巴拉克·奥巴马在拜登的 CNN 采访中的五大看点…… - 雅虎, link: https://www.usatoday.com/story/news/politics/2023/06/23/five-takeaways-from-barack-obama-cnn-interview/70349112007/], [snippet: 美国和世界各地的民主机构“变得不堪一击”，前总统巴拉克·奥巴马在一次独家 CNN 采访中警告说，现任总统特朗普……, title: 奥巴马警告民主机构“变得不堪一击”，但特朗普…… - CNN, link: https://www.cnn.com/2023/06/22/politics/barack-obama-interview-cnntv/index.html], [snippet: 巴拉克·奥巴马是美国第 44 任总统，也是第一位黑人总司令。他连任两届，从 2009 年到 2017 年。作为来自肯尼亚和堪萨斯的父母的儿子，奥巴马……, title: 巴拉克·奥巴马：传记，第 44 任美国总统，政治家, link: https://www.biography.com/political-figures/barack-obama], [snippet: 奥巴马基金会副总裁娜塔莉·布基-贝克（Natalie Bookey-Baker）曾在白宫为时任第一夫人米歇尔·奥巴马工作，她表示预计将有约 2500 名校友参加。他们是奥巴马……, title: 本周芝加哥举行巴拉克·奥巴马团队重聚活动；预计将有 2500 名校友参加……, link: https://chicago.suntimes.com/politics/2023/10/29/23937504/barack-obama-michelle-obama-david-axelrod-pod-save-america-jon-batiste-jen-psaki-reunion-obamaworld], [snippet: 2023 年 8 月 2 日，太平洋夏令时间下午 5:00。迈克·梅莫利和克里斯汀·韦尔克。华盛顿 —— 在 6 月份访问白宫期间，前总统巴拉克·奥巴马向他的前竞选搭档明确表示……, title: 奥巴马私下告诉拜登，他将尽一切努力帮助 2024 年, link: https://www.nbcnews.com/politics/white-house/obama-privately-told-biden-whatever-takes-help-2024-rcna97865]'
```
```python
您还可以直接将自定义的``DuckDuckGoSearchAPIWrapper``传递给``DuckDuckGoSearchResults``。因此，您可以更多地控制搜索结果。
```python

from langchain_community.utilities import DuckDuckGoSearchAPIWrapper

wrapper = DuckDuckGoSearchAPIWrapper(region="de-de", time="d", max_results=2)

```
```python
search = DuckDuckGoSearchResults(api_wrapper=wrapper, source="news")
```
```python
search.run("Obama")
```
```output

'[snippet: 当奥巴马于2017年1月离任时，CNN的一项民意调查显示他的支持率为60%，使他成为离任时总统支持率排名靠前的人之一。, title: 观点：特朗普攻击奥巴马医改的真正原因 | CNN, link: https://www.cnn.com/2023/12/04/opinions/trump-obamacare-obama-repeal-health-care-obeidallah/index.html], [snippet: 奥巴马的书单推荐。这部时长约两个小时的Netflix电影改编自备受赞誉的美国作家Rumaan Alam的同名小说《Leave the World Behind》。2020年，该作品入围了“国家图书奖”的决赛名单。在德国，这本书也被列入了备受关注的书单，其中包括了奥巴马的阅读推荐..., title: Netflix新片《Leave The World Behind》：对...的评论 - Prisma, link: https://www.prisma.de/news/filme/Neu-bei-Netflix-Leave-The-World-Behind-Kritik-zum-ungewoehnlichen-Endzeit-Film-mit-Julia-Roberts,46563944]'

```

