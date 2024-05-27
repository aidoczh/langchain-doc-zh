# 新闻网址

这篇文章介绍了如何从一系列网址中加载 HTML 新闻文章，并将其转换成我们可以在下游使用的文档格式。

```python
from langchain_community.document_loaders import NewsURLLoader
```

```python
urls = [
    "https://www.bbc.com/news/world-us-canada-66388172",
    "https://www.bbc.com/news/entertainment-arts-66384971",
]
```

将网址传入以加载它们到文档中

```python
loader = NewsURLLoader(urls=urls)
data = loader.load()
print("第一篇文章：", data[0])
print("\n第二篇文章：", data[1])
```

```output
第一篇文章： page_content='在对审查1月6日骚乱的国会委员会作证时，鲍威尔夫人表示，她并没有审查她提出的许多选举舞弊指控，她告诉他们，“没有一个理智的人”会把她的指控视为事实。她和她的代表都没有发表评论。' metadata={'title': '唐纳德·特朗普起诉：我们对六名共谋者了解多少？', 'link': 'https://www.bbc.com/news/world-us-canada-66388172', 'authors': [], 'language': 'en', 'description': '检察官描述了帮助特朗普先生破坏选举的六人。', 'publish_date': None}
第二篇文章： page_content='威廉姆斯女士补充说：“如果有任何事情我可以做到，以确保舞者或歌手或任何决定与她合作的人不必经历同样的经历，我会这样做。”' metadata={'title': 'Lizzo 舞者 Arianna Davis 和 Crystal Williams：‘没有人站出来，他们很害怕’', 'link': 'https://www.bbc.com/news/entertainment-arts-66384971', 'authors': [], 'language': 'en', 'description': '这位美国流行歌手因性骚扰和羞辱肥胖而被起诉，但她尚未发表评论。', 'publish_date': None}
```

使用 nlp=True 运行自然语言处理分析并生成关键词和摘要

```python
loader = NewsURLLoader(urls=urls, nlp=True)
data = loader.load()
print("第一篇文章：", data[0])
print("\n第二篇文章：", data[1])
```

```output
第一篇文章： page_content='在对审查1月6日骚乱的国会委员会作证时，鲍威尔夫人表示，她并没有审查她提出的许多选举舞弊指控，她告诉他们，“没有一个理智的人”会把她的指控视为事实。她和她的代表都没有发表评论。' metadata={'title': '唐纳德·特朗普起诉：我们对六名共谋者了解多少？', 'link': 'https://www.bbc.com/news/world-us-canada-66388172', 'authors': [], 'language': 'en', 'description': '检察官描述了帮助特朗普先生破坏选举的六人。', 'publish_date': None, 'keywords': ['powell', 'know', 'donald', 'trump', 'review', 'indictment', 'telling', 'view', 'reasonable', 'person', 'testimony', 'coconspirators', 'riot', 'representatives', 'claims'], 'summary': '在对审查1月6日骚乱的国会委员会作证时，鲍威尔夫人表示，她并没有审查她提出的许多选举舞弊指控，她告诉他们，“没有一个理智的人”会把她的指控视为事实。\n她和她的代表都没有发表评论。'}
第二篇文章： page_content='威廉姆斯女士补充说：“如果有任何事情我可以做到，以确保舞者或歌手或任何决定与她合作的人不必经历同样的经历，我会这样做。”' metadata={'title': 'Lizzo 舞者 Arianna Davis 和 Crystal Williams：‘没有人站出来，他们很害怕’', 'link': 'https://www.bbc.com/news/entertainment-arts-66384971', 'authors': [], 'language': 'en', 'description': '这位美国流行歌手因性骚扰和羞辱肥胖而被起诉，但她尚未发表评论。', 'publish_date': None, 'keywords': ['davis', 'lizzo', 'singers', 'experience', 'crystal', 'ensure', 'arianna', 'theres', 'williams', 'power', 'going', 'dancers', 'im', 'speaks', 'work', 'ms', 'scared'], 'summary': '威廉姆斯女士补充说：“如果有任何事情我可以做到，以确保舞者或歌手或任何决定与她合作的人不必经历同样的经历，我会这样做。”'}
```

```python
data[0].metadata["keywords"]
```

```output
['powell',
 'know',
 'donald',
 'trump',
 'review',
 'indictment',
 'telling',
 'view',
 'reasonable',
 'person',
 'testimony',
 'coconspirators',
 'riot',
 'representatives',
 'claims']
```

```python
data[0].metadata["summary"]
```

```output
'在对审查1月6日骚乱的国会委员会作证时，鲍威尔夫人表示，她并没有审查她提出的许多选举舞弊指控，她告诉他们，“没有一个理智的人”会把她的指控视为事实。\n她和她的代表都没有发表评论。'
```