# Tavily 搜索 API

[Tavily 的搜索 API](https://tavily.com) 是专门为 AI 代理人（LLMs）构建的搜索引擎，能够以实时、准确和事实为基础的速度提供结果。

我们可以将其用作[检索器](/docs/how_to#retrievers)。它将展示特定于此集成的功能。在浏览完后，探索[相关的用例页面](/docs/how_to#qa-with-rag)可能会有所帮助，以了解如何将此向量存储作为更大链条的一部分使用。

## 设置

该集成位于 `langchain-community` 包中。我们还需要安装 `tavily-python` 包本身。

```bash
pip install -U langchain-community tavily-python
```

我们还需要设置我们的 Tavily API 密钥。

```python
import getpass
import os
os.environ["TAVILY_API_KEY"] = getpass.getpass()
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

```python
from langchain_community.retrievers import TavilySearchAPIRetriever
retriever = TavilySearchAPIRetriever(k=3)
retriever.invoke("what year was breath of the wild released?")
```
```output
[Document(page_content='Trending topics\nTrending topics\nThe Legend of Zelda™: Breath of the Wild\nSelect a product\nThe Legend of Zelda™: Breath of the Wild\nThe Legend of Zelda™: Breath of the Wild\nThe Legend of Zelda™: Breath of the Wild and The Legend of Zelda™: Breath of the Wild Expansion Pass Bundle\nThis item will be sent to your system automatically after purchase or Nintendo Switch Game Voucher redemption. The Legend of Zelda: Breath of the Wild Expansion Pass\nMore like this\nSuper Mario Odyssey™\nThe Legend of Zelda™: Tears of the Kingdom\nMario + Rabbids® Kingdom Battle\nThe Legend of Zelda™: Link’s Awakening\nHollow Knight\nThe Legend of Zelda™: Skyward Sword HD\nStarlink: Battle for Atlas™ Digital Edition\nDRAGON QUEST BUILDERS™ 2\nDragon Quest Builders™\nWARNING: If you have epilepsy or have had seizures or other unusual reactions to flashing lights or patterns, consult a doctor before playing video games. Saddle up with a herd of horse-filled games!\nESRB rating\nSupported play modes\nTV\nTabletop\nHandheld\nProduct information\nRelease date\nNo. of players\nGenre\nPublisher\nESRB rating\nSupported play modes\nGame file size\nSupported languages\nPlay online, access classic NES™ and Super NES™ games, and more with a Nintendo Switch Online membership.\n Two Game Boy games are now available for Nintendo Switch Online members\n02/01/23\nNintendo Switch Online member exclusive: Save on two digital games\n09/13/22\nOut of the Shadows … the Legend of Zelda: About Nintendo\nShop\nMy Nintendo Store orders\nSupport\nParents\nCommunity\nPrivacy\n© Nintendo.', metadata={'title': 'The Legend of Zelda™: Breath of the Wild - Nintendo', 'source': 'https://www.nintendo.com/us/store/products/the-legend-of-zelda-breath-of-the-wild-switch/', 'score': 0.97451, 'images': None}),
 Document(page_content='The Legend of Zelda: Breath of the Wild is a masterpiece of open-world design and exploration, released on March 3, 2017 for Nintendo Switch. Find out the latest news, reviews, guides, videos, and more for this award-winning game on IGN.', metadata={'title': 'The Legend of Zelda: Breath of the Wild - IGN', 'source': 'https://www.ign.com/games/the-legend-of-zelda-breath-of-the-wild', 'score': 0.94496, 'images': None}),
 Document(page_content='Reviewers also commented on the unexpected permutations of interactions between Link, villagers, pets, and enemies,[129][130][131] many of which were shared widely on social media.[132] A tribute to former Nintendo president Satoru Iwata, who died during development, also attracted praise.[129][134]\nJim Sterling was more critical than most, giving Breath of the Wild a 7/10 score, criticizing the difficulty, weapon durability, and level design, but praising the open world and variety of content.[135] Other criticism focused on the unstable frame rate and the low resolution of 900p;[136] updates addressed some of these problems.[137][138]\nSales\nBreath of the Wild broke sales records for a Nintendo launch game in multiple regions.[139][140] In Japan, the Switch and Wii U versions sold a combined 230,000 copies in the first week of release, with the Switch version becoming the top-selling game released that week.[141] Nintendo reported that Breath of the Wild sold more than one million copies in the US that month—925,000 of which were for Switch, outselling the Switch itself.[145][146][147][148] Nintendo president Tatsumi Kimishima said that the attach rate on the Switch was "unprecedented".[149] Breath of the Wild had sold 31.15 million copies on the Switch by September 2023 and 1.70 million copies on the Wii U by December 2020.[150][151]\nAwards\nFollowing its demonstration at E3 2016,
## 链接
我们可以很容易地将这个检索器组合成一个链条。
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.
Context: {context}
Question: {question}"""
)
chain = (
    RunnablePassthrough.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | ChatOpenAI(model="gpt-4-1106-preview")
    | StrOutputParser()
)
```
```python
chain.invoke({"question": "how many units did bretch of the wild sell in 2020"})
```
```output

'截至2020年底，《塞尔达传说：荒野之息》全球销量超过2145万份。'

```