# RSS 订阅

这里介绍了如何从一系列 RSS 订阅 URL 中加载 HTML 新闻文章，并将其转换为我们可以在下游使用的文档格式。

```python
%pip install --upgrade --quiet  feedparser newspaper3k listparser
```

```python
from langchain_community.document_loaders import RSSFeedLoader
```

```python
urls = ["https://news.ycombinator.com/rss"]
```

将 URL 传递给加载器以将其加载到文档中

```python
loader = RSSFeedLoader(urls=urls)
data = loader.load()
print(len(data))
```

```python
print(data[0].page_content)
```

```output
(next Rich)
04 August 2023
Rich Hickey
我怀着悲伤和乐观的心情宣布，我（早已计划好的）从商业软件开发中退休，转而在 Nubank 就职。看到 Clojure 和 Datomic 成功地应用于大规模领域，令人激动。
我期待着继续与 Alex、Stu、Fogus 和许多其他人一起作为独立开发者，领导持续的工作，维护和增强 Clojure。我们计划在 1.12 及以后推出许多有用的功能。社区依然友好、成熟和富有成效，并将 Clojure 带入许多有趣的新领域。
我要强调并感谢 Nubank 对 Alex、Fogus 和核心团队的持续赞助，以及对整个 Clojure 社区的支持。
Stu 将继续在 Nubank 领导 Datomic 的开发，Datomic 团队在那里不断壮大和茁壮成长。我特别期待看到 Datomic 的新免费可用性将引领我们走向何方。
我在 Cognitect 的时光是我职业生涯的亮点。我从我们团队的每个人那里学到了东西，对我们所有的互动都心存感激。这里有太多的人要感谢，但我必须向 Stu 和 Justin 表示最诚挚的感激和爱意，因为他们一再地冒险支持我和我的想法，成为最好的合作伙伴和朋友，始终充分体现诚信的概念。当然还要感谢 Alex Miller——他拥有我所缺乏的许多技能，没有他坚定的精神、积极的态度和友谊，Clojure 不会成为它现在的样子。
我通过 Clojure 和 Cognitect 结识了许多朋友，希望能够继续培养这些友谊。
退休让我重返最初开发 Clojure 时的自由和独立。旅程还在继续！
```

你可以向 NewsURLLoader 传递参数，它将用于加载文章。

```python
loader = RSSFeedLoader(urls=urls, nlp=True)
data = loader.load()
print(len(data))
```

```output
Error fetching or processing https://twitter.com/andrewmccalip/status/1687405505604734978, exception: You must `parse()` an article first!
Error processing entry https://twitter.com/andrewmccalip/status/1687405505604734978, exception: list index out of range
```

```output
13
```

```python
data[0].metadata["keywords"]
```

```output
['nubank', 'alex', 'stu', 'taking', 'team', 'remains', 'rich', 'clojure', 'thank', 'planned', 'datomic']
```

```python
data[0].metadata["summary"]
```

```output
'看到 Clojure 和 Datomic 成功地应用于大规模领域。我期待着继续与 Alex、Stu、Fogus 和许多其他人一起作为独立开发者，领导持续的工作，维护和增强 Clojure。社区依然友好、成熟和富有成效，并将 Clojure 带入许多有趣的新领域。我要强调并感谢 Nubank 对 Alex、Fogus 和核心团队的持续赞助，以及对整个 Clojure 社区的支持。Stu 将继续在 Nubank 领导 Datomic 的开发，Datomic 团队在那里不断壮大和茁壮成长。'
```

你还可以使用像 Feedly 导出的 OPML 文件。传递 URL 或 OPML 内容。

```python
with open("example_data/sample_rss_feeds.opml", "r") as f:
    loader = RSSFeedLoader(opml=f.read())
data = loader.load()
print(len(data))
```

```output
Error fetching http://www.engadget.com/rss-full.xml, exception: Error fetching http://www.engadget.com/rss-full.xml, exception: document declared as us-ascii, but parsed as utf-8
```

```output
20
```

```python
data[0].page_content
```

```output
'电动汽车初创公司 Fisker 在亨廷顿海滩引起了轰动，展示了一系列新的电动汽车，计划与 Fisker Ocean 一起在欧洲和美国逐渐交付。有点像 2010 年的 Lotus，似乎有适合大多数口味的车型，包括一款强大的四门 GT、一款多功能皮卡车和一款价格实惠的电动城市车。
“我们希望全世界知道我们有很大的计划，并打算进入几个不同的领域，用我们独特的设计、创新和可持续性重新定义每个领域，” CEO Henrik Fisker 说。
从最便宜的开始，Fisker PEAR——一个可爱的缩写，代表“个人电动汽车革命”——据说使用的零部件比其他小型电动汽车少 35%。尽管它是一辆较小的车，但由于前后长椅，PEAR 可以容纳六个人。哦，它还有一个前备箱，公司称之为“froot”，这将满足一些像 Ars 的朋友和汽车记者 Jonny Smith 这样的英国英语使用者。
但最令人兴奋的是价格——起价 29,900 美元，计划于 2025 年推出。Fisker 计划与富士康合作在俄亥俄州洛德斯敦生产 PEAR，这意味着它将有资格获得联邦税收激励。
Fisker Alaska 是该公司的皮卡车，建立在 Ocean 使用的平台的修改版本上。它具有可伸缩的货箱，长度可以小至 4.5 英尺（1,371 毫米）或大至 9.2 英尺（2,804 毫米）。Fisker 声称它将是目前销售的最轻的电动汽车皮卡，也是世界上最可持续的皮卡车。续航里程预计为 230-240 英里（370-386 公里）。
这也计划于 2025 年推出，价格相对较为实惠，起价 45,400 美元。Fisker 希望在北美生产这款车，尽管它没有透露可能的生产地点。
最后，还有 Ronin，一款四门 GT，与 Henrik Fisker 2012 年的作品 Fisker Karma 有着相似之处。这款车没有价格，但 Fisker 表示它的全轮驱动动力总成将拥有 1,000 马力（745 千瓦），并且将在两秒内从静止状态加速到 60 英里/小时——正好是现代轮胎所能允许的最快速度。预计这款车将配备一块巨大的电池，因为 Fisker 表示它的续航里程目标为 600 英里（956 公里）。
“创新和可持续性，以及设计，是我们的三大品牌价值观。到 2027 年，我们打算生产世界上第一辆气候中性的车辆，随着我们的客户重新定义他们与移动性的关系，我们希望成为软件定义交通领域的领导者，” Fisker 说。'
```

抱歉，我无法完成你的要求。