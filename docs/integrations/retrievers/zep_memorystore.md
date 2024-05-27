# Zep

## [Zep](https://docs.getzep.com/) 的检索器示例

> 回忆、理解并提取聊天历史中的数据。为个性化的 AI 体验提供动力。

> [Zep](https://www.getzep.com) 是面向 AI 助手应用的长期记忆服务。

> 使用 Zep，您可以为 AI 助手提供回忆过去对话的能力，无论多久之前，

> 同时还可以减少幻觉、延迟和成本。

> 感兴趣 Zep 云？请参阅 [Zep 云安装指南](https://help.getzep.com/sdks) 和 [Zep 云检索器示例](https://help.getzep.com/langchain/examples/rag-message-history-example)

## 开源安装和设置

> Zep 开源项目：[https://github.com/getzep/zep](https://github.com/getzep/zep)

> Zep 开源文档：[https://docs.getzep.com/](https://docs.getzep.com/)

## 检索器示例

这个笔记本演示了如何使用[Zep 长期记忆存储](https://getzep.github.io/)搜索历史聊天消息记录。

我们将演示：

1. 将对话历史添加到 Zep 存储中。

2. 对对话历史进行向量搜索：

    1. 通过聊天消息进行相似性搜索

    2. 使用最大边际相关性重新排列聊天消息搜索结果

    3. 使用元数据过滤器过滤搜索结果

    4. 通过聊天消息摘要进行相似性搜索

    5. 使用最大边际相关性重新排列摘要搜索结果

```python
import getpass
import time
from uuid import uuid4
from langchain.memory import ZepMemory
from langchain_core.messages import AIMessage, HumanMessage
# 将此设置为您的 Zep 服务器 URL
ZEP_API_URL = "http://localhost:8000"
```

### 初始化 Zep 聊天消息历史类并将聊天消息历史添加到存储中

**注意：** 与其他检索器不同，Zep 检索器返回的内容是特定于会话/用户的。在实例化检索器时需要 `session_id`。

```python
# 提供您的 Zep API 密钥。请注意，这是可选的。请参阅 https://docs.getzep.com/deployment/auth
AUTHENTICATE = False
zep_api_key = None
if AUTHENTICATE:
    zep_api_key = getpass.getpass()
```
```python
session_id = str(uuid4())  # 这是用户/会话的唯一标识符
# 初始化 Zep 存储类
zep_memory = ZepMemory(session_id=session_id, url=ZEP_API_URL, api_key=zep_api_key)
```
```python
# 预加载一些消息到存储中。默认消息窗口为 12 条消息。我们希望超过这个数量以演示自动摘要。
test_history = [
    {"role": "human", "content": "奥克塔维亚·巴特勒是谁？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·埃斯特尔·巴特勒（1947年6月22日 - 2006年2月24日）是一位美国"
            " 科幻作家。"
        ),
    },
    {"role": "human", "content": "她的哪些书被改编成了电影？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒作品中最著名的改编作品是 FX 系列"
            "《血脉》，改编自她的同名小说。"
        ),
    },
    {"role": "human", "content": "她的同时代人是谁？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒的同时代人包括乌苏拉·勒·瑰恩、塞缪尔·R·"
            " 德莱尼和乔安娜·拉斯。"
        ),
    },
    {"role": "human", "content": "她获得了哪些奖项？"},
    {
        "role": "ai",
        "content": (
            "奥克塔维亚·巴特勒获得了雨果奖、星云奖和麦克阿瑟"
            " 研究奖。"
        ),
    },
    {
        "role": "human",
        "content": "我可能想读哪些其他女性科幻作家的书？",
    },
    {
        "role": "ai",
        "content": "您可能想读乌苏拉·勒·瑰恩或乔安娜·拉斯的作品。",
    },
    {
        "role": "human",
        "content": (
            "为巴特勒的书《播种者寓言》写一个简短的梗概。它讲述了什么？"
        ),
    },
    {
        "role": "ai",
        "content": (
            "《播种者寓言》是奥克塔维亚·巴特勒的一部科幻小说，"
            " 出版于1993年。它讲述了年轻女子劳伦·奥拉米娜的故事，她生活在一个由于"
            " 环境灾难、贫困和暴力而崩溃的未来社会中。"
        ),
    },
    {"role": "human", "content": "这本书的背景是什么？"},
    {
        "role": "ai",
        "content": (
            "这本书设定在2020年代的一个崩溃的未来社会，原因是"
            " 气候变化和经济危机。"
        ),
    },
    {"role": "human", "content": "谁是主角？"},
    {
        "role": "ai",
        "content": (
            "这本书的主角是劳伦·奥拉米娜，一个年轻女子，她拥有"
            " '超共情'，即能感受她目睹的痛苦和其他感觉。"
        ),
    },
    {"role": "human", "content": "这本书的主题是什么？"},
    {
        "role": "ai",
        "content": (
            "这本书的主题是在急剧社会变革和崩溃中的生存。它还探讨了适应性、社区的主题，"
            " 以及人类对变化的能力。"
        ),
    },
    {"role": "human", "content": "《播种者寓言》是什么？"},
    {
        "role": "ai",
        "content": (
            "《播种者寓言》是巴特勒在书中使用的一则圣经寓言的隐喻。在寓言中，播种者撒种，"
            " 有些落在肥沃的土地上并生长，而其他的落在岩石地或荆棘中而无法生长。"
            " 寓言用来说明在面对变化时接受性和准备性的重要性。"
        ),
    },
    {"role": "human", "content": "巴特勒的写作风格如何？"},
    {
        "role": "ai",
        "content": (
            "巴特勒的写作风格以其清晰、直接和心理洞察力而闻名。她的叙事经常涉及复杂、多样化"
            " 的角色，并探讨种族、性别和权力的主题。"
        ),
    },
    {"role": "human", "content": "她还写了哪些书？"},
    {
        "role": "ai",
        "content": (
            "除了《播种者寓言》，巴特勒还写了几部其他著名作品，包括《血脉》、《黎明》和《播种者的天赋》。"
        ),
    },
]
for msg in test_history:
    zep_memory.chat_memory.add_message(
        HumanMessage(content=msg["content"])
        if msg["role"] == "human"
        else AIMessage(content=msg["content"])
    )
time.sleep(
    10
)  # 等待消息被嵌入和摘要。速度取决于 OpenAI API 的延迟和您的速率限制。
### 使用 Zep Retriever 进行 Zep 存储器的向量搜索
Zep 提供了对历史对话存储器的本地向量搜索。嵌入会自动进行。
注意：消息的嵌入是异步进行的，因此第一次查询可能不会返回结果。随着嵌入的生成，后续查询将返回结果。
```python

from langchain_community.retrievers.zep import SearchScope, SearchType, ZepRetriever

zep_retriever = ZepRetriever(

    session_id=session_id,  # 确保在实例化检索器时提供 session_id

    url=ZEP_API_URL,

    top_k=5,

    api_key=zep_api_key,

)

await zep_retriever.ainvoke("谁写了《播种者的寓言》？")

```
```output

[Document(page_content="What is the 'Parable of the Sower'?", metadata={'score': 0.9250216484069824, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),

 Document(page_content='Parable of the Sower is a science fiction novel by Octavia Butler, published in 1993. It follows the story of Lauren Olamina, a young woman living in a dystopian future where society has collapsed due to environmental disasters, poverty, and violence.', metadata={'score': 0.8897348046302795, 'uuid': '3dd9f5ed-9dc9-4427-9da6-aba1b8278a5c', 'created_at': '2023-11-01T00:32:40.192527Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'GPE', 'Matches': [{'End': 20, 'Start': 15, 'Text': 'Sower'}], 'Name': 'Sower'}, {'Label': 'PERSON', 'Matches': [{'End': 65, 'Start': 51, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'DATE', 'Matches': [{'End': 84, 'Start': 80, 'Text': '1993'}], 'Name': '1993'}, {'Label': 'PERSON', 'Matches': [{'End': 124, 'Start': 110, 'Text': 'Lauren Olamina'}], 'Name': 'Lauren Olamina'}], 'intent': 'Providing information'}}, 'token_count': 56}),

 Document(page_content="Write a short synopsis of Butler's book, Parable of the Sower. What is it about?", metadata={'score': 0.8856019973754883, 'uuid': '81761dcb-38f3-4686-a4f5-6cb1007eaf29', 'created_at': '2023-11-01T00:32:40.187543Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'ORG', 'Matches': [{'End': 32, 'Start': 26, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 61, 'Start': 41, 'Text': 'Parable of the Sower'}], 'Name': 'Parable of the Sower'}], 'intent': "The subject is asking for a brief summary of Butler's book, Parable of the Sower, and what it is about."}}, 'token_count': 23}),

 Document(page_content="The 'Parable of the Sower' is a biblical parable that Butler uses as a metaphor in the book. In the parable, a sower scatters seeds, some of which fall on fertile ground and grow, while others fall on rocky ground or among thorns and fail to grow. The parable is used to illustrate the importance of receptivity and preparedness in the face of change.", metadata={'score': 0.8781436681747437, 'uuid': '1a8c5f99-2fec-425d-bc37-176ab91e7080', 'created_at': '2023-11-01T00:32:40.22836Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 26, 'Start': 5, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 60, 'Start': 54, 'Text': 'Butler'}], 'Name': 'Butler'}]}}, 'token_count': 84}),

 Document(page_content="In addition to 'Parable of the Sower', Butler has written several other notable works, including 'Kindred', 'Dawn', and 'Parable of the Talents'.", metadata={'score': 0.8745182752609253, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39})]

```
我们还可以使用 Zep 同步 API 来检索结果：
```python
zep_retriever.invoke("谁写了《播种者的寓言》？")
```
```output```
### 什么是《撒种人的比喻》？
《撒种人的比喻》是奥克塔维亚·巴特勒（Octavia Butler）于1993年出版的一部科幻小说。故事主要讲述了一个名叫劳伦·奥拉米娜（Lauren Olamina）的年轻女性，在一个由环境灾难、贫困和暴力导致社会崩溃的反乌托邦未来中生活的情节。[20]
巴特勒在这本书中使用《撒种人的比喻》作为隐喻。在这个比喻中，一个撒种的人撒下种子，有些落在肥沃的土地上生长，而其他的落在岩石或荆棘中而无法生长。这个比喻被用来说明在面对变化时接受和准备的重要性。[21]
此外，巴特勒还写了其他几部著名作品，包括《血缘》（Kindred）、《黎明》（Dawn）和《撒种人的比喻》（Parable of the Talents）。[22]
### 使用元数据过滤器来精确搜索结果
Zep支持使用元数据来过滤搜索结果。这对于按实体类型或其他元数据进行过滤非常有用。
更多信息请参考：[https://docs.getzep.com/sdk/search_query/](https://docs.getzep.com/sdk/search_query/)
```python
filter = {"where": {"jsonpath": '$[*] ? (@.Label == "WORK_OF_ART")'}}
await zep_retriever.ainvoke("谁写了《播种者的寓言》？", metadata=filter)
```
```output

[Document(page_content="《播种者的寓言》是什么？", metadata={'score': 0.9251098036766052, 'uuid': '4cbfb1c0-6027-4678-af43-1e18acb224bb', 'created_at': '2023-11-01T00:32:40.224256Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 34, 'Start': 13, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}]}}, 'token_count': 13}),

 Document(page_content='她还写了哪些其他书？', metadata={'score': 0.7747920155525208, 'uuid': '1b3c5079-9cab-46f3-beae-fb56c572e0fd', 'created_at': '2023-11-01T00:32:40.240135Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'human', 'token_count': 9}),

 Document(page_content="除了《播种者的寓言》外，巴特勒还写了其他几部知名作品，包括《血缘》、《黎明》和《才能的寓言》。", metadata={'score': 0.8745266795158386, 'uuid': '45d8aa08-85ab-432f-8902-81712fe363b9', 'created_at': '2023-11-01T00:32:40.245081Z', 'updated_at': '0001-01-01T00:00:00Z', 'role': 'ai', 'metadata': {'system': {'entities': [{'Label': 'WORK_OF_ART', 'Matches': [{'End': 37, 'Start': 16, 'Text': "Parable of the Sower'"}], 'Name': "Parable of the Sower'"}, {'Label': 'ORG', 'Matches': [{'End': 45, 'Start': 39, 'Text': 'Butler'}], 'Name': 'Butler'}, {'Label': 'GPE', 'Matches': [{'End': 105, 'Start': 98, 'Text': 'Kindred'}], 'Name': 'Kindred'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 144, 'Start': 121, 'Text': "Parable of the Talents'"}], 'Name': "Parable of the Talents'"}]}}, 'token_count': 39}),

```
### 使用 MMR 重新排名的摘要搜索
Zep 自动生成聊天消息的摘要。这些摘要可以使用 Zep Retriever 进行搜索。由于摘要是对对话的精炼，它们更有可能匹配您的搜索查询，并为 LLM 提供丰富而简洁的上下文。
连续的摘要可能包含相似的内容，Zep 的相似性搜索返回最匹配的结果，但多样性较小。
MMR 重新排名结果，以确保您填充到提示中的摘要既相关，又提供额外信息给 LLM。
```python
zep_retriever = ZepRetriever(
    session_id=session_id,  # 确保在实例化检索器时提供 session_id
    url=ZEP_API_URL,
    top_k=3,
    api_key=zep_api_key,
    search_scope=SearchScope.summary,
    search_type=SearchType.mmr,
    mmr_lambda=0.5,
)
await zep_retriever.ainvoke("谁写了《播种者寓言》？")
```
```output

[Document(page_content='人类询问有关 Octavia Butler，AI 告知他们她是一位美国科幻小说作家。人类询问她的哪些书被改编成电影，AI 提到了 FX 系列《Kindred》。人类随后询问她的同时代作家，AI 列举了 Ursula K. Le Guin、Samuel R. Delany 和 Joanna Russ。人类还询问她获得的奖项，AI 提到了雨果奖、星云奖和麦克阿瑟基金会奖。人类询问其他女性科幻作家推荐，AI 建议了 Ursula K. Le Guin 和 Joanna Russ。人类随后要求对 Butler 的书《播种者寓言》进行概要描述。', metadata={'score': 0.7882999777793884, 'uuid': '3c95a29a-52dc-4112-b8a7-e6b1dc414d45', 'created_at': '2023-11-01T00:32:47.76449Z', 'token_count': 155}),

 Document(page_content='人类询问有关 Octavia Butler。AI 告知人类 Octavia Estelle Butler 是一位美国科幻小说作家。人类随后询问她的哪些书被改编成电影，AI 提到了 FX 系列《Kindred》，该系列基于她的同名小说。', metadata={'score': 0.7407922744750977, 'uuid': '0e027f4d-d71f-42ae-977f-696b8948b8bf', 'created_at': '2023-11-01T00:32:41.637098Z', 'token_count': 59}),

 Document(page_content='人类询问有关 Octavia Butler，AI 告知他们她是一位美国科幻小说作家。人类询问她的哪些书被改编成电影，AI 提到了 FX 系列《Kindred》。人类随后询问她的同时代作家，AI 列举了 Ursula K. Le Guin、Samuel R. Delany 和 Joanna Russ。人类还询问她获得的奖项，AI 提到了雨果奖、星云奖和麦克阿瑟基金会奖。', metadata={'score': 0.7436535358428955, 'uuid': 'b3500d1b-1a78-4aef-9e24-6b196cfa83cb', 'created_at': '2023-11-01T00:32:44.24744Z', 'token_count': 104})]

```