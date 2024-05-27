# 勇敢搜索

[勇敢搜索](https://en.wikipedia.org/wiki/Brave_Search) 是由勇敢软件开发的搜索引擎。

- `勇敢搜索` 使用自己的网络索引。截至2022年5月，它覆盖了超过100亿个页面，并且在92%的搜索结果中不依赖任何第三方，其余结果从Bing API服务器端检索或（根据用户选择）从Google客户端检索。根据勇敢公司的说法，该索引被故意保持“比Google或Bing的索引小”，以帮助避免垃圾信息和其他低质量内容，但缺点是“勇敢搜索在恢复长尾查询方面还不如Google”。

- `勇敢搜索高级版`：截至2023年4月，勇敢搜索是一个无广告的网站，但最终将转向一个新模式，其中将包括广告，高级用户将享受无广告体验。用户数据，包括IP地址，默认情况下不会被收集。选择性数据收集将需要高级账户。

## 安装和设置

要访问勇敢搜索API，您需要[创建一个账户并获取API密钥](https://api.search.brave.com/app/dashboard)。

```python
api_key = "..."
```

```python
from langchain_community.document_loaders import BraveSearchLoader
```

## 示例

```python
loader = BraveSearchLoader(
    query="obama middle name", api_key=api_key, search_kwargs={"count": 3}
)
docs = loader.load()
len(docs)
```

```output
3
```

```python
[doc.metadata for doc in docs]
```

```output
[{'title': "Obama's Middle Name -- My Last Name -- is 'Hussein.' So?",
  'link': 'https://www.cair.com/cair_in_the_news/obamas-middle-name-my-last-name-is-hussein-so/'},
 {'title': "What's up with Obama's middle name? - Quora",
  'link': 'https://www.quora.com/Whats-up-with-Obamas-middle-name'},
 {'title': 'Barack Obama | Biography, Parents, Education, Presidency, Books, ...',
  'link': 'https://www.britannica.com/biography/Barack-Obama'}]
```

```python
[doc.page_content for doc in docs]
```

```output
['我不确定几天前听广播脱口秀主持人比尔·坎宁安（Bill Cunningham）反复尖叫巴拉克·<strong>奥巴马</strong>的<strong>中间</strong><strong>名</strong>——我的姓氏——就好像他患有反穆斯林的特发性症候群一样。坎宁安嘶嘶作响地说“Hussein”，就像在呼唤撒旦一样...',
 '回答（15个回答中的第1个）：一个更好的问题应该是，“奥巴马的<strong>名字</strong>是怎么回事？”美国总统巴拉克·侯赛因·<strong>奥巴马</strong>的父亲的<strong>名字</strong>是巴拉克·侯赛因·<strong>奥巴马</strong>。他是以他父亲的名字命名的。侯赛因，奥巴马的<strong>中间</strong><strong>名字</strong>，是一个非常常见的阿拉伯<strong>名字</strong>，意思是“好”，“英俊”，或...',
 '巴拉克·<strong>奥巴马</strong>，全名巴拉克·侯赛因·<strong>奥巴马</strong>二世（1961年8月4日出生于美国夏威夷州檀香山），是美国第44任总统（2009年至2017年）和第一位担任此职务的非裔美国人。在当选总统之前，<strong>奥巴马</strong>代表伊利诺伊州参加美国...']
```