# ChatPerplexity

这篇笔记介绍了如何开始使用 Perplexity 聊天模型。

```python
from langchain_community.chat_models import ChatPerplexity
from langchain_core.prompts import ChatPromptTemplate
```

提供的代码假设你的 PPLX_API_KEY 已设置在环境变量中。如果你想手动指定 API 密钥并选择不同的模型，可以使用以下代码：

```python
chat = ChatPerplexity(temperature=0, pplx_api_key="YOUR_API_KEY", model="pplx-70b-online")
```

你可以在[这里](https://docs.perplexity.ai/docs/model-cards)查看可用模型的列表。为了可重现性，我们可以通过在本笔记本中将其作为输入来动态设置 API 密钥。

```python
import os
from getpass import getpass
PPLX_API_KEY = getpass()
os.environ["PPLX_API_KEY"] = PPLX_API_KEY
```

```python
chat = ChatPerplexity(temperature=0, model="pplx-70b-online")
```

```python
system = "You are a helpful assistant."
human = "{input}"
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
chain = prompt | chat
response = chain.invoke({"input": "Why is the Higgs Boson important?"})
response.content
```

```output
'希格斯玻色子是一种基本的亚原子粒子，在粒子物理标准模型中起着至关重要的作用，该模型解释了宇宙行为的四种基本力量中的三种：强力、弱力、电磁力和引力。希格斯玻色子之所以重要有几个原因：
1. **最终的基本粒子**：希格斯玻色子是标准模型下等待被发现的最后一个基本粒子。其探测有助于完善标准模型，并进一步增进我们对宇宙中基本力量的理解。
2. **质量生成**：希格斯玻色子负责赋予其他粒子质量，这一过程是通过其与希格斯场的相互作用而发生的。这种质量生成对于原子、分子和我们在宇宙中观察到的可见物质的形成至关重要。
3. **对新物理的启示**：虽然希格斯玻色子的探测已经证实了标准模型的许多方面，但它也为标准模型之外的新发现开辟了新的可能性。对希格斯玻色子的进一步研究可能揭示出有关暗物质、超对称性和其他奇特现象的见解。
4. **技术的进步**：寻找希格斯玻色子导致了技术的重大进步，例如在粒子加速器（如大型强子对撞机）中使用的人工智能和机器学习算法的发展。这些进步不仅有助于发现希格斯玻色子，还可能在其他各个领域有潜在的应用。
总之，希格斯玻色子之所以重要是因为它完善了标准模型，对质量生成起着至关重要的作用，暗示着标准模型之外的新物理现象，并推动了技术的进步。'
```

你可以像通常一样格式化和构造提示。在下面的例子中，我们要求模型给我们讲一个关于猫的笑话。

```python
chat = ChatPerplexity(temperature=0, model="pplx-70b-online")
prompt = ChatPromptTemplate.from_messages([("human", "Tell me a joke about {topic}")])
chain = prompt | chat
response = chain.invoke({"topic": "cats"})
response.content
```

```output
'这是一个关于猫的笑话：
为什么猫想从美人鱼那里学数学？
因为它找不到自己的“核心目标”！
记住，猫是独特而迷人的生物，每只猫都有自己独特的特点和能力。虽然有些人可能认为它们神秘，甚至有点高傲，但它们仍然是深受人们喜爱的宠物，给主人带来快乐和陪伴。所以，如果你的猫曾经向美人鱼寻求指导，只要记住它们正在自我发现的旅程中！'
```

## `ChatPerplexity` 还支持流式功能：

```python
chat = ChatPerplexity(temperature=0.7, model="pplx-70b-online")
prompt = ChatPromptTemplate.from_messages([("human", "Give me a list of famous tourist attractions in Pakistan")])
chain = prompt | chat
for chunk in chain.stream({}):
    print(chunk.content, end="", flush=True)
```

```output
以下是巴基斯坦一些著名旅游景点的列表：
1. **巴基斯坦纪念碑**：位于伊斯兰堡，代表着巴基斯坦的四个省和三个地区。
2. **巴德沙希清真寺**：位于拉合尔的历史悠久的清真寺，可容纳1万名礼拜者。
3. **沙利玛花园**：位于拉合尔的美丽花园，拥有精心设计的庭院和一系列倾泻的水池。
4. **巴基斯坦国家博物馆**：位于卡拉奇的博物馆，展示了该国的文化历史。
5. **法萨尔清真寺**：位于伊斯兰堡的大型清真寺，可容纳30万名礼拜者。
6. **克利夫顿海滩**：位于卡拉奇的热门海滩，提供水上活动和休闲设施。
7. **卡塔尔普尔走廊**：连接巴基斯坦的达巴尔萨希布古尔德瓦拉和印度的古鲁达拉斯希布的无签证边境通道和宗教走廊。
8. **摩亨佐-达罗**：位于巴基斯坦信德省的古代印度河流域文明遗址，可追溯到公元前2500年左右。
9. **洪扎山谷**：位于吉尔吉特-巴尔蒂斯坦的风景如画的山谷，以其壮丽的山景和独特的文化而闻名。
这些景点展示了巴基斯坦丰富的历史、多样的文化和自然美景，使它们成为当地和国际游客的热门目的地。
```