# Upstash Vector

> [Upstash Vector](https://upstash.com/docs/vector/overall/whatisvector) 是一个为向量嵌入工作而设计的无服务器向量数据库。

> vector langchain 集成是 [upstash-vector](https://github.com/upstash/vector-py) 包的一个封装。

> 该 Python 包在后台使用 [vector rest api](https://upstash.com/docs/vector/api/get-started)。

## 安装

从 [upstash 控制台](https://console.upstash.com/vector) 创建一个免费的向量数据库，设置所需的维度和距离度量。

然后，可以通过以下方式创建一个 `UpstashVectorStore` 实例：

- 提供环境变量 `UPSTASH_VECTOR_URL` 和 `UPSTASH_VECTOR_TOKEN`

- 将它们作为参数传递给构造函数

- 将一个 Upstash Vector `Index` 实例传递给构造函数

此外，还需要一个 `Embeddings` 实例将给定的文本转换为嵌入向量。这里我们以 `OpenAIEmbeddings` 为例。

```python
%pip install langchain-openai langchain upstash-vector
```

```python
import os
from langchain_community.vectorstores.upstash import UpstashVectorStore
from langchain_openai import OpenAIEmbeddings
os.environ["OPENAI_API_KEY"] = "<YOUR_OPENAI_KEY>"
os.environ["UPSTASH_VECTOR_REST_URL"] = "<YOUR_UPSTASH_VECTOR_URL>"
os.environ["UPSTASH_VECTOR_REST_TOKEN"] = "<YOUR_UPSTASH_VECTOR_TOKEN>"
# 创建一个嵌入实例
embeddings = OpenAIEmbeddings()
# 创建一个向量存储实例
store = UpstashVectorStore(embedding=embeddings)
```

创建 `UpstashVectorStore` 的另一种方法是通过选择一个模型来 [创建一个 Upstash Vector 索引](https://upstash.com/docs/vector/features/embeddingmodels#using-a-model)，并传递 `embedding=True`。在这种配置下，文档或查询将作为文本发送到 Upstash 并在那里进行嵌入。

```python
store = UpstashVectorStore(embedding=True)
```

如果您有兴趣尝试这种方法，可以像上面那样更新 `store` 的初始化，并运行本教程的其余部分。

## 加载文档

加载一个示例文本文件，并将其拆分为可以转换为向量嵌入的块。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
docs[:3]
```

```output
[Document(page_content='Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  \n\nLast year COVID-19 kept us apart. This year we are finally together again. \n\nTonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. \n\nWith a duty to one another to the American people to the Constitution. \n\nAnd with an unwavering resolve that freedom will always triumph over tyranny. \n\nSix days ago, Russia’s Vladimir Putin sought to shake the foundations of the free world thinking he could make it bend to his menacing ways. But he badly miscalculated. \n\nHe thought he could roll into Ukraine and the world would roll over. Instead he met a wall of strength he never imagined. \n\nHe met the Ukrainian people. \n\nFrom President Zelenskyy to every Ukrainian, their fearlessness, their courage, their determination, inspires the world.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 Document(page_content='Groups of citizens blocking tanks with their bodies. Everyone from students to retirees teachers turned soldiers defending their homeland. \n\nIn this struggle as President Zelenskyy said in his speech to the European Parliament “Light will win over darkness.” The Ukrainian Ambassador to the United States is here tonight. \n\nLet each of us here tonight in this Chamber send an unmistakable signal to Ukraine and to the world. \n\nPlease rise if you are able and show that, Yes, we the United States of America stand with the Ukrainian people. \n\nThroughout our history we’ve learned this lesson when dictators do not pay a price for their aggression they cause more chaos.   \n\nThey keep moving.   \n\nAnd the costs and the threats to America and the world keep rising.   \n\nThat’s why the NATO Alliance was created to secure peace and stability in Europe after World War 2. \n\nThe United States is a member along with 29 other nations. \n\nIt matters. American diplomacy matters. American resolve matters.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 Document(page_content='Putin’s latest attack on Ukraine was premeditated and unprovoked. \n\nHe rejected repeated efforts at diplomacy. \n\nHe thought the West and NATO wouldn’t respond. And he thought he could divide us at home. Putin was wrong. We were ready.  Here is what we did.   \n\nWe prepared extensively and carefully. \n\nWe spent months building a coalition of other freedom-loving nations from Europe and the Americas to Asia and Africa to confront Putin. \n\nI spent countless hours unifying our European allies. We shared with the world in advance what we knew Putin was planning and precisely how he would try to falsely justify his aggression.  \n\nWe countered Russia’s lies with truth.   \n\nAnd now that he has acted the free world is holding him accountable. \n\nAlong with twenty-seven members of the European Union including France, Germany, Italy, as well as countries like the United Kingdom, Canada, Japan, Korea, Australia, New Zealand, and many others, even Switzerland.', metadata={'source': '../../how_to/state_of_the_union.txt'})]
```

## 插入文档

Vectorstore使用嵌入对象将文本块嵌入并批量插入到数据库中。这将返回一个已插入向量的ID数组。

```python
inserted_vectors = store.add_documents(docs)
inserted_vectors[:5]
```

```output
['82b3781b-817c-4a4d-8f8b-cbd07c1d005a',
 'a20e0a49-29d8-465e-8eae-0bc5ac3d24dc',
 'c19f4108-b652-4890-873e-d4cad00f1b1a',
 '23d1fcf9-6ee1-4638-8c70-0f5030762301',
 '2d775784-825d-4627-97a3-fee4539d8f58']
```

store

```python
store.add_texts(
    [
        "A timeless tale set in the Jazz Age, this novel delves into the lives of affluent socialites, their pursuits of wealth, love, and the elusive American Dream. Amidst extravagant parties and glittering opulence, the story unravels the complexities of desire, ambition, and the consequences of obsession.",
        "Set in a small Southern town during the 1930s, this novel explores themes of racial injustice, moral growth, and empathy through the eyes of a young girl. It follows her father, a principled lawyer, as he defends a black man accused of assaulting a white woman, confronting deep-seated prejudices and challenging societal norms along the way.",
        "A chilling portrayal of a totalitarian regime, this dystopian novel offers a bleak vision of a future world dominated by surveillance, propaganda, and thought control. Through the eyes of a disillusioned protagonist, it explores the dangers of totalitarianism and the erosion of individual freedom in a society ruled by fear and oppression.",
        "Set in the English countryside during the early 19th century, this novel follows the lives of the Bennet sisters as they navigate the intricate social hierarchy of their time. Focusing on themes of marriage, class, and societal expectations, the story offers a witty and insightful commentary on the complexities of romantic relationships and the pursuit of happiness.",
        "Narrated by a disillusioned teenager, this novel follows his journey of self-discovery and rebellion against the phoniness of the adult world. Through a series of encounters and reflections, it explores themes of alienation, identity, and the search for authenticity in a society marked by conformity and hypocrisy.",
        "In a society where emotion is suppressed and individuality is forbidden, one man dares to defy the oppressive regime. Through acts of rebellion and forbidden love, he discovers the power of human connection and the importance of free will.",
        "Set in a future world devastated by environmental collapse, this novel follows a group of survivors as they struggle to survive in a harsh, unforgiving landscape. Amidst scarcity and desperation, they must confront moral dilemmas and question the nature of humanity itself.",
    ],
    [
        {"title": "The Great Gatsby", "author": "F. Scott Fitzgerald", "year": 1925},
        {"title": "To Kill a Mockingbird", "author": "Harper Lee", "year": 1960},
        {"title": "1984", "author": "George Orwell", "year": 1949},
        {"title": "Pride and Prejudice", "author": "Jane Austen", "year": 1813},
        {"title": "The Catcher in the Rye", "author": "J.D. Salinger", "year": 1951},
        {"title": "Brave New World", "author": "Aldous Huxley", "year": 1932},
        {"title": "The Road", "author": "Cormac McCarthy", "year": 2006},
    ],
)
```

```output
['fe1f7a7b-42e2-4828-88b0-5b449c49fe86',
 '154a0021-a99c-427e-befb-f0b2b18ed83c',
 'a8218226-18a9-4ab5-ade5-5a71b19a7831',
 '62b7ef97-83bf-4b6d-8c93-f471796244dc',
 'ab43fd2e-13df-46d4-8cf7-e6e16506e4bb',
 '6841e7f9-adaa-41d9-af3d-0813ee52443f',
 '45dda5a1-f0c1-4ac7-9acb-50253e4ee493']
```

## 查询

可以使用向量或文本提示查询数据库。

如果使用文本提示，首先将其转换为嵌入，然后进行查询。

`k`参数指定从查询中返回多少结果。

```python
result = store.similarity_search("The United States of America", k=5)
result
```

```output
[Document(page_content='And my report is this: the State of the Union is strong—because you, the American people, are strong. \n\nWe are stronger today than we were a year ago. \n\nAnd we will be stronger a year from now than we are today. \n\nNow is our moment to meet and overcome the challenges of our time. \n\nAnd we will, as one people. \n\nOne America. \n\nThe United States of America. \n\nMay God bless you all. May God protect our troops.', metadata={'source': '../../how_to/state_of_the_union.txt'}),```

建立了世界上最强大、最自由、最繁荣的国家。

现在是时候了。

我们的责任之时。

我们的决心和良知的考验，历史本身的考验。

正是在这一刻，我们的品格形成了。我们的目标被找到了。我们的未来被锻造了。

我很了解这个国家。

我们将通过考验。

保护自由和自由，扩大公平和机会。

我们将拯救民主。

尽管这些时期是多么艰难，但我对今天的美国比我一生都更加乐观。

因为我看到了我们可以掌握的未来。

因为我知道没有什么是我们无法做到的。

我们是地球上唯一一个将我们面临的每一次危机都转化为机遇的国家。

唯一一个可以用一个词来定义的国家：可能性。

所以在这个夜晚，在我们作为一个国家的第245个年头，我来报告国情。

公民们组成的团体用自己的身体阻挡坦克。从学生到退休教师，他们变成了保卫家园的士兵。

正如泽连斯基总统在他在欧洲议会的演讲中所说的那样，“光明将战胜黑暗”。乌克兰驻美国大使今晚在这里。

让我们今晚在这个议会大厅里的每一个人向乌克兰和世界发出一个明确的信号。

如果你能站起来，请站起来，表明我们美利坚合众国与乌克兰人民站在一起。

在我们的历史上，我们学到了这个教训，当独裁者不为他们的侵略付出代价时，他们会造成更多的混乱。

他们会继续前进。

而对美国和世界的威胁和成本也会不断上升。

这就是为什么北约联盟在二战后成立，以确保欧洲的和平与稳定。

美国是其中的一员，还有其他29个国家。

这很重要。美国的外交很重要。美国的决心很重要。

当我们使用纳税人的钱来重建美国时，我们将购买美国货：购买美国产品以支持美国就业。

联邦政府每年花费约6000亿美元来确保国家的安全。

几乎有一个世纪的法律确保纳税人的钱支持美国的就业和企业。

每届政府都说他们会这样做，但我们确实在做。

我们将购买美国货，确保从航空母舰的甲板到公路护栏上的钢材都是美国制造的。

但为了争取未来最好的工作，我们还需要与中国和其他竞争对手公平竞争。

这就是为什么通过坐在国会中的两党创新法案如此重要，该法案将在新兴技术和美国制造业方面进行创纪录的投资。

让我给你举一个例子，说明为什么通过这个法案如此重要。

女士们，先生们，我们的第一夫人和第二绅士。国会议员和内阁成员。最高法院法官。我的美国同胞们。

去年，COVID-19让我们分开。今年，我们终于再次聚在一起。

今晚，我们作为民主党人、共和党人和独立人士相聚在一起。但更重要的是，作为美国人。

我们有责任对彼此、对美国人民、对宪法。

我们坚定不移地相信，自由将永远战胜暴政。

六天前，俄罗斯的弗拉基米尔·普京试图动摇自由世界的基础，以为他可以使其屈服于他的威胁方式。但他严重错误估计了。

他以为他可以闯入乌克兰，世界会屈服。相反，他遇到了他从未想象过的强大力量的墙壁。

他遇到了乌克兰人民。

从泽连斯基总统到每一个乌克兰人，他们的无畏、勇气和决心激励着世界。

## 带有分数的查询

每个结果都可以包含查询的分数。

> 查询请求中返回的分数是一个标准化值，介于0和1之间，其中1表示最高相似度，0表示最低相似度，无论使用的相似度函数如何。有关更多信息，请参阅[文档](https://upstash.com/docs/vector/overall/features#vector-similarity-functions)。

```python
result = store.similarity_search_with_score("美利坚合众国", k=5)
for doc, score in result:
    print(f"{doc.metadata} - {score}")
```

```output
{'source': '../../how_to/state_of_the_union.txt'} - 0.87391514
{'source': '../../how_to/state_of_the_union.txt'} - 0.8549463
{'source': '../../how_to/state_of_the_union.txt'} - 0.847913
{'source': '../../how_to/state_of_the_union.txt'} - 0.84328896
{'source': '../../how_to/state_of_the_union.txt'} - 0.832347
```

## 删除向量

可以通过其ID删除向量。

```python
store.delete(inserted_vectors)
```

## 清除向量数据库

这将清除向量数据库。

```python
store.delete(delete_all=True)
```

## 获取有关向量数据库的信息

您可以使用info函数获取有关数据库的信息，例如距离度量维度。

> 当发生插入时，数据库会进行索引。在此过程中，无法查询新向量。`pendingVectorCount`表示当前正在进行索引的向量数量。

```python
store.info()
```

```output
InfoResult(vector_count=42, pending_vector_count=0, index_size=6470, dimension=384, similarity_function='COSINE')
```