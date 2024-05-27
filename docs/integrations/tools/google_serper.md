# 谷歌搜索器

本文介绍如何使用 `Google Serper` 组件来搜索网络。首先，您需要在 [serper.dev](https://serper.dev) 注册一个免费账户并获取您的 API 密钥。

```python
import os
import pprint
os.environ["SERPER_API_KEY"] = ""
```
```python
from langchain_community.utilities import GoogleSerperAPIWrapper
```
```python
search = GoogleSerperAPIWrapper()
```
```python
search.run("奥巴马的名字是什么？")
```
```output
'巴拉克·侯赛因·奥巴马二世'
```

## 作为自问自答链的一部分

```python
os.environ["OPENAI_API_KEY"] = ""
```
```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.utilities import GoogleSerperAPIWrapper
from langchain_core.tools import Tool
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
search = GoogleSerperAPIWrapper()
tools = [
    Tool(
        name="Intermediate Answer",
        func=search.run,
        description="useful for when you need to ask with search",
    )
]
self_ask_with_search = initialize_agent(
    tools, llm, agent=AgentType.SELF_ASK_WITH_SEARCH, verbose=True
)
self_ask_with_search.run(
    "现任男子美国公开赛冠军的家乡是哪里？"
)
```
```output
> 进入新的 AgentExecutor 链...
 是的。
后续问题：谁是现任男子美国公开赛冠军？
中间答案：卡洛斯·阿尔卡拉斯，2022年男子单打冠军。
后续问题：卡洛斯·阿尔卡拉斯来自哪里？
中间答案：西班牙的埃尔帕尔马。
因此最终答案是：西班牙的埃尔帕尔马
> 链结束。
```
```output
'西班牙的埃尔帕尔马'
```

## 获取带有元数据的结果

如果您还希望以结构化的方式获取结果，包括元数据。我们将使用包装器的 `results` 方法。

```python
search = GoogleSerperAPIWrapper()
results = search.results("苹果公司")
pprint.pp(results)
```
```output
{'searchParameters': {'q': 'Apple Inc.',
                      'gl': 'us',
                      'hl': 'en',
                      'num': 10,
                      'type': 'search'},
 'knowledgeGraph': {'title': 'Apple',
                    'type': 'Technology company',
                    'website': 'http://www.apple.com/',
                    'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQwGQRv5TjjkycpctY66mOg_e2-npacrmjAb6_jAWhzlzkFE3OTjxyzbA&s=0',
                    'description': 'Apple Inc. is an American multinational '
                                   'technology company headquartered in '
                                   'Cupertino, California. Apple is the '
                                   "world's largest technology company by "
                                   'revenue, with US$394.3 billion in 2022 '
                                   'revenue. As of March 2023, Apple is the '
                                   "world's biggest...",
                    'descriptionSource': 'Wikipedia',
                    'descriptionLink': 'https://en.wikipedia.org/wiki/Apple_Inc.',
                    'attributes': {'Customer service': '1 (800) 275-2273',
                                   'CEO': 'Tim Cook (Aug 24, 2011–)',
                                   'Headquarters': 'Cupertino, CA',
                                   'Founded': 'April 1, 1976, Los Altos, CA',
                                   'Founders': 'Steve Jobs, Steve Wozniak, '
                                               'Ronald Wayne, and more',
                                   'Products': 'iPhone, iPad, Apple TV, and '
                                               'more'}},
 'organic': [{'title': 'Apple',
              'link': 'https://www.apple.com/',
              'snippet': 'Discover the innovative world of Apple and shop '
                         'everything iPhone, iPad, Apple Watch, Mac, and Apple '
                         'TV, plus explore accessories, entertainment, ...',
              'sitelinks': [{'title': 'Support',
                             'link': 'https://support.apple.com/'},
                            {'title': 'iPhone',
                             'link': 'https://www.apple.com/iphone/'},
                            {'title': 'Site Map',
                             'link': 'https://www.apple.com/sitemap/'},
                            {'title': 'Business',
                             'link': 'https://www.apple.com/business/'},
                            {'title': 'Mac',
                             'link': 'https://www.apple.com/mac/'},
                            {'title': 'Watch',
                             'link': 'https://www.apple.com/watch/'}],
              'position': 1},
             {'title': 'Apple Inc. - Wikipedia',
              'link': 'https://en.wikipedia.org/wiki/Apple_Inc.',
              'snippet': 'Apple Inc. is an American multinational technology '
                         'company headquartered in Cupertino, California. '
                         "Apple is the world's largest technology company by "
                         'revenue, ...',
              'attributes': {'Products': 'AirPods; Apple Watch; iPad; iPhone; '
                                         'Mac; Full list',
                             'Founders': 'Steve Jobs; Steve Wozniak; Ronald '
                                         'Wayne; Mike Markkula'},
              'sitelinks': [{'title': 'History',
                             'link': 'https://en.wikipedia.org/wiki/History_of_Apple_Inc.'},
                            {'title': 'Timeline of Apple Inc. products',
                             'link': 'https://en.wikipedia.org/wiki/Timeline_of_Apple_Inc._products'},
                            {'title': 'Litigation involving Apple Inc.',
                             'link': 'https://en.wikipedia.org/wiki/Litigation_involving_Apple_Inc.'},
                            {'title': 'Apple Store',
                             'link': 'https://en.wikipedia.org/wiki/Apple_Store'}],
              'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRvmB5fT1LjqpZx02UM7IJq0Buoqt0DZs_y0dqwxwSWyP4PIN9FaxuTea0&s',
              'position': 2},
             {'title': 'Apple Inc. | History, Products, Headquarters, & Facts '
                       '| Britannica',
              'link': 'https://www.britannica.com/topic/Apple-Inc',
              'snippet': 'Apple Inc., formerly Apple Computer, Inc., American '
                         'manufacturer of personal computers, smartphones, '
                         'tablet computers, computer peripherals, and computer '
                         '...',
              'attributes': {'Related People': 'Steve Jobs Steve Wozniak Jony '
                                               'Ive Tim Cook Angela Ahrendts'},
              'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS3liELlhrMz3Wpsox29U8jJ3L8qETR0hBWHXbFnwjwQc34zwZvFELst2E&s',
              'position': 3},
             {'title': 'AAPL: Apple Inc Stock Price Quote - NASDAQ GS - '
                       'Bloomberg.com',
              'link': 'https://www.bloomberg.com/quote/AAPL:US',
              'snippet': 'AAPL:USNASDAQ GS. Apple Inc. COMPANY INFO ; Open. '
                         '170.09 ; Prev Close. 169.59 ; Volume. 48,425,696 ; '
                         'Market Cap. 2.667T ; Day Range. 167.54170.35.',
              'position': 4},
             {'title': 'Apple Inc. (AAPL) Company Profile & Facts - Yahoo '
                       'Finance',
              'link': 'https://finance.yahoo.com/quote/AAPL/profile/',
              'snippet': 'Apple Inc. designs, manufactures, and markets '
                         'smartphones, personal computers, tablets, wearables, '
                         'and accessories worldwide. The company offers '
                         'iPhone, a line ...',
              'position': 5},
             {'title': 'Apple Inc. (AAPL) Stock Price, News, Quote & History - '
                       'Yahoo Finance',
              'link': 'https://finance.yahoo.com/quote/AAPL',
              'snippet': 'Find the latest Apple Inc. (AAPL) stock quote, '
                         'history, news and other vital information to help '
                         'you with your stock trading and investing.',
              'position': 6}],
 'peopleAlsoAsk': [{'question': 'Apple Inc. 做什么？',
                    'snippet': 'Apple Inc. (Apple) designs, manufactures and '
                               'markets smartphones, personal\n'
                               'computers, tablets, wearables and accessories '
                               'and sells a range of related\n'
                               'services.',
                    'title': 'AAPL.O - | Stock Price & Latest News - Reuters',
                    'link': 'https://www.reuters.com/markets/companies/AAPL.O/'},
                   {'question': 'Apple Inc 的全称是什么？',
                    'snippet': '(formerly Apple Computer Inc.) is an American '
                               'computer and consumer electronics\n'
                               'company famous for creating the iPhone, iPad '
                               'and Macintosh computers.',
                    'title': 'What is Apple? An products and history overview '
                             '- TechTarget',
                    'link': 'https://www.techtarget.com/whatis/definition/Apple'},
                   {'question': 'Apple Inc iPhone 是什么？',
                    'snippet': 'Apple Inc (Apple) designs, manufactures, and '
                               'markets smartphones, tablets,\n'
                               'personal computers, and wearable devices. The '
                               'company also offers software\n'
                               'applications and related services, '
                               'accessories, and third-party digital content.\n'
                               "Apple's product portfolio includes iPhone, "
                               'iPad, Mac, iPod, Apple Watch, and\n'
                               'Apple TV.',
                    'title': 'Apple Inc Company Profile - Apple Inc Overview - '
                             'GlobalData',
                    'link': 'https://www.globaldata.com/company-profile/apple-inc/'},
                   {'question': '谁管理 Apple Inc？',
                    'snippet': 'Timothy Donald Cook (born November 1, 1960) is '
                               'an American business executive\n'
                               'who has been the chief executive officer of '
                               'Apple Inc. since 2011. Cook\n'
                               "previously served as the company's chief "
                               'operating officer under its co-founder\n'
                               'Steve Jobs. He is the first CEO of any Fortune '
                               '500 company who is openly gay.',
                    'title': 'Tim Cook - Wikipedia',
                    'link': 'https://en.wikipedia.org/wiki/Tim_Cook'}],
 'relatedSearches': [{'query': '谁发明了 iPhone'},
                     {'query': '苹果 iPhone'},
                     {'query': '苹果公司历史 PDF'},
                     {'query': '苹果公司历史'},
                     {'query': '苹果公司介绍'},
                     {'query': '苹果印度'},
                     {'query': '苹果公司拥有什么'},
                     {'query': '史蒂夫离开后的苹果公司'},
                     {'query': '苹果手表'},
                     {'query': '苹果应用商店'}]}
```

## 搜索谷歌图片

我们也可以使用这个包装器来查询谷歌图片。例如：

```python
search = GoogleSerperAPIWrapper(type="images")
results = search.results("狮子")
pprint.pp(results)
```
```output
{'searchParameters': {'q': 'Lion',
                      'gl': 'us',
                      'hl': 'en',
                      'num': 10,
                      'type': 'images'},
 'images': [{'title': 'Lion - Wikipedia',
             'imageUrl': 'https://upload.wikimedia.org/wikipedia/commons/thumb/7/73/Lion_waiting_in_Namibia.jpg/1200px-Lion_waiting_in_Namibia.jpg',
             'imageWidth': 1200,
             'imageHeight': 900,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRye79ROKwjfb6017jr0iu8Bz2E1KKuHg-A4qINJaspyxkZrkw&amp;s',
             'thumbnailWidth': 259,
             'thumbnailHeight': 194,
             'source': 'Wikipedia',
             'domain': 'en.wikipedia.org',
             'link': 'https://en.wikipedia.org/wiki/Lion',
             'position': 1},
            {'title': 'Lion | Characteristics, Habitat, & Facts | Britannica',
             'imageUrl': 'https://cdn.britannica.com/55/2155-050-604F5A4A/lion.jpg',
             'imageWidth': 754,
             'imageHeight': 752,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS3fnDub1GSojI0hJ-ZGS8Tv-hkNNloXh98DOwXZoZ_nUs3GWSd&amp;s',
             'thumbnailWidth': 225,
             'thumbnailHeight': 224,
             'source': 'Encyclopedia Britannica',
             'domain': 'www.britannica.com',
             'link': 'https://www.britannica.com/animal/lion',
             'position': 2},
            {'title': 'African lion, facts and photos',
             'imageUrl': 'https://i.natgeofe.com/n/487a0d69-8202-406f-a6a0-939ed3704693/african-lion.JPG',
             'imageWidth': 3072,
             'imageHeight': 2043,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTPlTarrtDbyTiEm-VI_PML9VtOTVPuDXJ5ybDf_lN11H2mShk&amp;s',
             'thumbnailWidth': 275,
             'thumbnailHeight': 183,
             'source': 'National Geographic',
             'domain': 'www.nationalgeographic.com',
             'link': 'https://www.nationalgeographic.com/animals/mammals/facts/african-lion',
             'position': 3},
            {'title': 'Saint Louis Zoo | African Lion',
             'imageUrl': 'https://optimise2.assets-servd.host/maniacal-finch/production/animals/african-lion-01-01.jpg?w=1200&auto=compress%2Cformat&fit=crop&dm=1658933674&s=4b63f926a0f524f2087a8e0613282bdb',
             'imageWidth': 1200,
             'imageHeight': 1200,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTlewcJ5SwC7yKup6ByaOjTnAFDeoOiMxyJTQaph2W_I3dnks4&amp;s',
             'thumbnailWidth': 225,
             'thumbnailHeight': 225,
             'source': 'St. Louis Zoo',
             'domain': 'stlzoo.org',
             'link': 'https://stlzoo.org/animals/mammals/carnivores/lion',
             'position': 4},
            {'title': 'How to Draw a Realistic Lion like an Artist - Studio Wildlife',
             'imageUrl': 'https://studiowildlife.com/wp-content/uploads/2021/10/245528858_183911853822648_6669060845725210519_n.jpg',
             'imageWidth': 1431,
             'imageHeight': 2048,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTmn5HayVj3wqoBDQacnUtzaDPZzYHSLKUlIEcni6VB8w0mVeA&amp;s',
             'thumbnailWidth': 188,
             'thumbnailHeight': 269,
             'source': 'Studio Wildlife',
             'domain': 'studiowildlife.com',
             'link': 'https://studiowildlife.com/how-to-draw-a-realistic-lion-like-an-artist/',
             'position': 5},
            {'title': 'Lion | Characteristics, Habitat, & Facts | Britannica',
             'imageUrl': 'https://cdn.britannica.com/29/150929-050-547070A1/lion-Kenya-Masai-Mara-National-Reserve.jpg',
             'imageWidth': 1600,
             'imageHeight': 1085,
             'thumbnailUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCqaKY_THr0IBZN8c-2VApnnbuvKmnsWjfrwKoWHFR9w3eN5o&amp;s',
             'thumbnailWidth': 273,
             'thumbnailHeight': 185,
             'source': 'Encyclopedia Britannica',
             'domain': 'www.britannica.com',
             'link': 'https://www.britannica.com/animal/lion',
             'position': 6},
            {'title': "Where do lions live? Facts about lions' habitats and other cool facts",
             'imageUrl': 'https://www.gannett-cdn.com/-mm-/b2b05a4ab25f4fca0316459e1c7404c537a89702/c=0-0-1365-768/local/-/media/2022/03/16/USATODAY/usatsports/imageForEntry5-ODq.jpg?width=1365&height=768&fit=crop&format=pjpg&auto=webp',
             'imageWidth': 1365,
             'imageHeight': 768,
```
```markdown
## 搜索谷歌新闻
我们也可以使用这个包装器查询谷歌新闻。例如：
```python
search = GoogleSerperAPIWrapper(type="news")
results = search.results("Tesla Inc.")
pprint.pp(results)
```
```output

{'searchParameters': {'q': 'Tesla Inc.',

                      'gl': 'us',

                      'hl': 'en',

                      'num': 10,

                      'type': 'news'},

 'news': [{'title': 'ISS recommends Tesla investors vote against re-election of Robyn Denholm',

           'link': 'https://www.reuters.com/business/autos-transportation/iss-recommends-tesla-investors-vote-against-re-election-robyn-denholm-2023-05-04/',

           'snippet': 'Proxy advisory firm ISS on Wednesday recommended Tesla investors vote against re-election of board chair Robyn Denholm, citing "concerns on...',

           'date': '5 mins ago',

           'source': 'Reuters',

           'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcROdETe_GUyp1e8RHNhaRM8Z_vfxCvdfinZwzL1bT1ZGSYaGTeOojIdBoLevA&s',

           'position': 1},

          {'title': 'Global companies by market cap: Tesla fell most in April',

           'link': 'https://www.reuters.com/markets/global-companies-by-market-cap-tesla-fell-most-april-2023-05-02/',

           'snippet': 'Tesla Inc was the biggest loser among top companies by market capitalisation in April, hit by disappointing quarterly earnings after it...',

           'date': '1 day ago',

           'source': 'Reuters',

           'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ4u4CP8aOdGyRFH6o4PkXi-_eZDeY96vLSag5gDjhKMYf98YBER2cZPbkStQ&s',

           'position': 2},

          {'title': 'Tesla Wanted an EV Price War. Ford Showed Up.',

           'link': 'https://www.bloomberg.com/opinion/articles/2023-05-03/tesla-wanted-an-ev-price-war-ford-showed-up',

           'snippet': 'The legacy automaker is paring back the cost of its Mustang Mach-E model after Tesla discounted its competing EVs, portending tighter...',

           'date': '6 hours ago',

           'source': 'Bloomberg.com',

           'imageUrl': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS_3Eo4VI0H-nTeIbYc5DaQn5ep7YrWnmhx6pv8XddFgNF5zRC9gEpHfDq8yQ&s',

           'position': 3},

```

# Tesla Inc. 获得投资，Joby Aviation 获得来自特斯拉股东 Baillie Gifford 的投资

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQO0uVn297LI-xryrPNqJ-apUOulj4ohM-xkN4OfmvMOYh1CPdUEBbYx6hviw&s)

这是在 Joby 赢得了价值 5500 万美元的合同延期之后的几天，该合同将交付多达九架空中出租车给美国空军... [20]

来源：Yahoo Finance

---

# 特斯拉恢复了美国低价、长续航里程 Model 3 版本的订单

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTIZetJ62sQefPfbQ9KKDt6iH7Mc0ylT5t_hpgeeuUkHhJuAx2FOJ4ZTRVDFg&s)

（路透社）- 特斯拉公司已经恢复了在美国接受其 Model 3 长续航车型的订单，公司网站显示... [21]

来源：Yahoo Finance

---

# 特斯拉 Model 3 长续航全轮驱动版现在在美国可用，续航里程为 325 英里

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSecrgxZpRj18xIJY-nDHljyP-A4ejEkswa9eq77qhMNrScnVIqe34uql5U4w&s)

特斯拉已经重新开放了 Model 3 长续航后驱版的订单，由于需求旺盛，该车型已经数月不可用。 [22]

来源：Not a Tesla App

---

# 特斯拉 Cybertruck alpha 原型在弗里蒙特工厂的新照片和视频中被发现

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRO7M5ZLQE-Zo4-_5dv9hNAQZ3wSqfvYCuKqzxHG-M6CgLpwPMMG_ssebdcMg&s)

特斯拉 Cybertruck alpha 原型在去年年底投产之前，前往加利福尼亚州弗里蒙特进行另一轮测试（图片... [23]

来源：Tesla Oracle

---

# 特斯拉在美国新地区设立工厂 - 奥斯汀商业杂志

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcR9kIEHWz1FcHKDUtGQBS0AjmkqtyuBkQvD8kyIY3kpaPrgYaN7I_H2zoOJsA&s)

查看普吉特湾商业杂志对这家总部位于奥斯汀的公司在太平洋西北地区房地产印记的报道。 [24]

来源：The Business Journals

---

# 特斯拉（TSLA）在订单积压后恢复了 Model 3 长续航版的订单

![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTWWIC4VpMTfRvSyqiomODOoLg0xhoBf-Tc1qweKnSuaiTk-Y1wMJZM3jct0w&s)

特斯拉公司已经恢复了其起价为 47240 美元的 Model 3 长续航版的订单，根据其网站显示。 [25]

来源：Bloomberg.com

## 谷歌搜索结果

在这个示例中，我们展示了谷歌搜索结果的一些片段。这些片段包括了标题、链接、摘要、日期、来源和图片链接等信息。这些信息可以帮助用户快速了解搜索结果的相关内容。

- 标题："T))，特斯拉公司（TSLA.O）的电动汽车供应商，周日表示正在考虑在俄克拉荷马州建造第三座电池工厂..."

- 日期："53分钟前"

- 来源："路透社"

- ![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSSTcsXeenqmEKdiekvUgAmqIPR4nlAmgjTkBqLpza-lLfjX1CwB84MoNVj0Q&s)

- 标题："Ryder在美国推出电动车一站式解决方案"

- 日期："56分钟前"

- 来源："《交通旅游杂志》"

- ![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQJhXTQQtjSUZf9YPM235WQhFU5_d7lEA76zB8DGwZfixcgf1_dhPJyKA1Nbw&s)

- 标题："“我认为人们可以用9.99亿美元过得很好”，伯尼·桑德斯告诉美国亿万富翁"

- 日期："11分钟前"

- 来源："《印度快报新闻》"

- ![图片](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcR_X9qqSwVFBBdos2CK5ky5IWIE3aJPCQeRYR9O1Jz4t-MjaEYBuwK7AU3AJQ&s)

这些搜索结果提供了关于特定主题的相关信息，包括新闻、事件和观点。用户可以通过这些搜索结果快速了解最新的信息和动态。

{'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipPJr2Vcx-B6K-GNQa4koOTffggTePz8TKRTnWi3=w92-h92-n-k-no',

 'rating': 4.5,

 'ratingCount': 344,

 'category': '意大利菜'},

{'position': 6,

 'title': 'Come Prima',

 'address': '903 Madison Ave',

 'latitude': 40.772124999999996,

 'longitude': -73.965012,

 'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipNrX19G0NVdtDyMovCQ-M-m0c_gLmIxrWDQAAbz=w92-h92-n-k-no',

 'rating': 4.5,

 'ratingCount': 176,

 'category': '意大利菜'},

{'position': 7,

 'title': 'Botte UES',

 'address': '1606 1st Ave.',

 'latitude': 40.7750785,

 'longitude': -73.9504801,

 'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipPPN5GXxfH3NDacBc0Pt3uGAInd9OChS5isz9RF=w92-h92-n-k-no',

 'rating': 4.4,

 'ratingCount': 152,

 'category': '意大利菜'},

{'position': 8,

 'title': 'Piccola Cucina Uptown',

 'address': '106 E 60th St',

 'latitude': 40.7632468,

 'longitude': -73.9689825,

 'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipPifIgzOCD5SjgzzqBzGkdZCBp0MQsK5k7M7znn=w92-h92-n-k-no',

 'rating': 4.6,

 'ratingCount': 941,

 'category': '意大利菜'},

{'position': 9,

 'title': 'Pinocchio Restaurant',

 'address': '300 E 92nd St',

 'latitude': 40.781453299999995,

 'longitude': -73.9486788,

 'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipNtxlIyEEJHtDtFtTR9nB38S8A2VyMu-mVVz72A=w92-h92-n-k-no',

 'rating': 4.5,

 'ratingCount': 113,

 'category': '意大利菜'},

{'position': 10,

 'title': 'Barbaresco',

 'address': '843 Lexington Ave #1',

 'latitude': 40.7654332,

 'longitude': -73.9656873,

 'thumbnailUrl': 'https://lh5.googleusercontent.com/p/AF1QipMb9FbPuXF_r9g5QseOHmReejxSHgSahPMPJ9-8=w92-h92-n-k-no',

 'rating': 4.3,

 'ratingCount': 122,

 'locationHint': '在The Touraine内',

 'category': '意大利菜'}]}