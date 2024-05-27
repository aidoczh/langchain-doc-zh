# Zep

> 回忆、理解并提取聊天记录中的数据。为个性化的 AI 体验提供动力。

[Zep](https://www.getzep.com) 是一种长期记忆服务，适用于 AI 助手应用程序。

使用 Zep，您可以让 AI 助手具备回忆过去对话的能力，无论过去多久，

同时还可以减少幻觉、延迟和成本。

对 Zep Cloud 感兴趣吗？请参阅 [Zep Cloud 安装指南](https://help.getzep.com/sdks) 和 [Zep Cloud 记忆示例](https://help.getzep.com/langchain/examples/messagehistory-example)

## 开源安装和设置

Zep 开源项目：[https://github.com/getzep/zep](https://github.com/getzep/zep)

Zep 开源文档：[https://docs.getzep.com/](https://docs.getzep.com/)

## 示例

这个笔记本演示了如何将 [Zep](https://www.getzep.com/) 用作聊天机器人的记忆。

REACT Agent Chat Message History with Zep - 用于 LLM 应用程序的长期记忆存储。

我们将演示：

1. 将对话历史添加到 Zep 中。

2. 运行代理并自动将消息添加到存储中。

3. 查看丰富的消息。

4. 在对话历史中进行向量搜索。

```python
from uuid import uuid4
from langchain.agents import AgentType, initialize_agent
from langchain.memory import ZepMemory
from langchain_community.retrievers import ZepRetriever
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_core.messages import AIMessage, HumanMessage
from langchain_core.tools import Tool
from langchain_openai import OpenAI
# 将此设置为您的 Zep 服务器 URL
ZEP_API_URL = "http://localhost:8000"
session_id = str(uuid4())  # 这是用户的唯一标识符
```

```python
# 提供您的 OpenAI 密钥
import getpass
openai_key = getpass.getpass()
```

```python
# 提供您的 Zep API 密钥。请注意，这是可选的。请参阅 https://docs.getzep.com/deployment/auth
zep_api_key = getpass.getpass()
```

### 初始化 Zep 聊天消息历史类并初始化代理

```python
search = WikipediaAPIWrapper()
tools = [
    Tool(
        name="Search",
        func=search.run,
        description=(
            "useful for when you need to search online for answers. You should ask"
            " targeted questions"
        ),
    ),
]
# 设置 Zep 聊天历史
memory = ZepMemory(
    session_id=session_id,
    url=ZEP_API_URL,
    api_key=zep_api_key,
    memory_key="chat_history",
)
# 初始化代理
llm = OpenAI(temperature=0, openai_api_key=openai_key)
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    verbose=True,
    memory=memory,
)
```

### 添加一些历史数据

```python
# 预加载一些消息到存储中。默认消息窗口为 12 条消息。我们希望超过这个数量以演示自动摘要。
test_history = [
    {"role": "human", "content": "Who was Octavia Butler?"},
    {
        "role": "ai",
        "content": (
            "Octavia Estelle Butler (June 22, 1947 – February 24, 2006) was an American"
            " science fiction author."
        ),
    },
    {"role": "human", "content": "Which books of hers were made into movies?"},
    {
        "role": "ai",
        "content": (
            "The most well-known adaptation of Octavia Butler's work is the FX series"
            " Kindred, based on her novel of the same name."
        ),
    },
    {"role": "human", "content": "Who were her contemporaries?"},
    {
        "role": "ai",
        "content": (
            "Octavia Butler's contemporaries included Ursula K. Le Guin, Samuel R."
            " Delany, and Joanna Russ."
        ),
    },
    {"role": "human", "content": "What awards did she win?"},
    {
        "role": "ai",
        "content": (
            "Octavia Butler won the Hugo Award, the Nebula Award, and the MacArthur"
            " Fellowship."
        ),
    },
    {
        "role": "human",
        "content": "Which other women sci-fi writers might I want to read?",
    },
    {
        "role": "ai",
        "content": "You might want to read Ursula K. Le Guin or Joanna Russ.",
    },
    {
        "role": "human",
        "content": (
            "Write a short synopsis of Butler's book, Parable of the Sower. What is it"
            " about?"
        ),
    },
    {
        "role": "ai",
        "content": (
            "Parable of the Sower is a science fiction novel by Octavia Butler,"
            " published in 1993. It follows the story of Lauren Olamina, a young woman"
            " living in a dystopian future where society has collapsed due to"
            " environmental disasters, poverty, and violence."
        ),
        "metadata": {"foo": "bar"},
    },
]
for msg in test_history:
    memory.chat_memory.add_message(
        (
            HumanMessage(content=msg["content"])
            if msg["role"] == "human"
            else AIMessage(content=msg["content"])
        ),
        metadata=msg.get("metadata", {}),
    )
```

### 运行代理程序

这样做将自动将输入和响应添加到 Zep 记忆中。

```python
agent_chain.run(
    input="What is the book's relevance to the challenges facing contemporary society?",
)
```

```output
> 进入新链...
思考：我需要使用工具吗？不需要
AI：《播种者的寓言》是一部有先见之明的小说，探讨了当代社会面临的挑战，如气候变化、不平等和暴力。这是一个告诫人们要警惕无节制的贪婪，需要个体对自己的生活和周围人的生活负起责任的寓言故事。
> 完成链。
```

```output
'《播种者的寓言》是一部有先见之明的小说，探讨了当代社会面临的挑战，如气候变化、不平等和暴力。这是一个告诫人们要警惕无节制的贪婪，需要个体对自己的生活和周围人的生活负起责任的寓言故事。'
```

### 检查 Zep 记忆

注意摘要，并且历史记录已经丰富了标记计数、UUID 和时间戳。

摘要偏向于最近的消息。

```python
def print_messages(messages):
    for m in messages:
        print(m.type, ":\n", m.dict())
print(memory.chat_memory.zep_summary)
print("\n")
print_messages(memory.chat_memory.messages)
```

```output
人类询问有关 Octavia Butler。AI 将她确定为美国科幻作家。人类随后询问她的作品中有哪些被改编成电影。AI 回答提到了基于她同名小说的 FX 系列《血缘》。人类随后询问她的同时代作家，AI 列举了厄休拉·K·勒·格恩、塞缪尔·R·戴拉尼和乔安娜·拉斯。
system :
 {'content': '人类询问有关 Octavia Butler。AI 将她确定为美国科幻作家。人类随后询问她的作品中有哪些被改编成电影。AI 回答提到了基于她同名小说的 FX 系列《血缘》。人类随后询问她的同时代作家，AI 列举了厄休拉·K·勒·格恩、塞缪尔·R·戴拉尼和乔安娜·拉斯。', 'additional_kwargs': {}}
human :
 {'content': '她赢得了哪些奖项？', 'additional_kwargs': {'uuid': '6b733f0b-6778-49ae-b3ec-4e077c039f31', 'created_at': '2023-07-09T19:23:16.611232Z', 'token_count': 8, 'metadata': {'system': {'entities': [], 'intent': '主题是询问某人（身份未指定）获得了哪些奖项。'}}}, 'example': False}
ai :
 {'content': 'Octavia Butler 赢得了雨果奖、星云奖和麦克阿瑟基金奖。', 'additional_kwargs': {'uuid': '2f6d80c6-3c08-4fd4-8d4e-7bbee341ac90', 'created_at': '2023-07-09T19:23:16.618947Z', 'token_count': 21, 'metadata': {'system': {'entities': [{'Label': 'PERSON', 'Matches': [{'End': 14, 'Start': 0, 'Text': 'Octavia Butler'}], 'Name': 'Octavia Butler'}, {'Label': 'WORK_OF_ART', 'Matches': [{'End': 33, 'Start': 19, 'Text': '雨果奖'}], 'Name': '雨果奖'}, {'Label': 'EVENT', 'Matches': [{'End': 81, 'Start': 57, 'Text': '麦克阿瑟基金奖'}], 'Name': '麦克阿瑟基金奖'}], 'intent': '主题是陈述 Octavia Butler 获得了雨果奖、星云奖和麦克阿瑟基金奖。'}}}, 'example': False}
human :
 {'content': '我可能想要阅读哪些其他女性科幻作家的作品？', 'additional_kwargs': {'uuid': 'ccdcc901-ea39-4981-862f-6fe22ab9289b', 'created_at': '2023-07-09T19:23:16.62678Z', 'token_count': 14, 'metadata': {'system': {'entities': [], 'intent': '主题是寻求推荐额外的女性科幻作家的作品。'}}}, 'example': False}
ai :
 {'content': '你可能想要阅读厄休拉·K·勒·格恩或乔安娜·拉斯的作品。', 'additional_kwargs': {'uuid': '7977099a-0c62-4c98-bfff-465bbab6c9c3', 'created_at': '2023-07-09T19:23:16.631721Z', 'token_count': 18, 'metadata': {'system': {'entities': [{'Label': 'ORG', 'Matches': [{'End': 40, 'Start': 23, 'Text': '厄休拉·K·勒·格恩'}], 'Name': '厄休拉·K·勒·格恩'}, {'Label': 'PERSON', 'Matches': [{'End': 55, 'Start': 44, 'Text': '乔安娜·拉斯'}], 'Name': '乔安娜·拉斯'}], 'intent': '主题是建议该人考虑阅读厄休拉·K·勒·格恩或乔安娜·拉斯的作品。'}}}, 'example': False}
human :
```

**巴特勒的《播种者寓言》**是一部科幻小说，由奥克塔维亚·巴特勒于1993年出版。故事主要讲述了一个名叫劳伦·奥拉米纳的年轻女性，在一个由环境灾难、贫困和暴力引发的末世未来中生活的情节。这部小说对当代社会面临的挑战，如气候变化、不平等和暴力等问题进行了预见性的探讨，警示人们对于无节制的贪婪的危险，并强调个体需要对自己的生活和周围人的生活负起责任。这部小说对于当代社会的挑战具有重要的现实意义。

在Zep的历史对话记忆中，可以通过`ZepRetriever`进行本地向量搜索。您可以使用支持传入Langchain `Retriever`对象的链式`ZepRetriever`。

```python
retriever = ZepRetriever(
    session_id=session_id,
    url=ZEP_API_URL,
    api_key=zep_api_key,
)
search_results = memory.chat_memory.search("who are some famous women sci-fi authors?")
for r in search_results:
    if r.dist > 0.8:  # 只打印相似度大于0.8的结果
        print(r.message, r.dist)
```

```output
{'uuid': 'ccdcc901-ea39-4981-862f-6fe22ab9289b', 'created_at': '2023-07-09T19:23:16.62678Z', 'role': 'human', 'content': 'Which other women sci-fi writers might I want to read?', 'metadata': {'system': {'entities': [], 'intent': 'The subject is seeking recommendations for additional women science fiction writers to explore.'}}, 'token_count': 14} 0.9119619869747062
{'uuid': '7977099a-0c62-4c98-bfff-465bbab6c9c3', 'created_at': '2023-07-09T19:23:16.631721Z', 'role': 'ai', 'content': 'You might want to read Ursula K. Le Guin or Joanna Russ.', 'metadata': {'system': {'entities': [{'Label': 'ORG', 'Matches': [{'End': 40, 'Start': 23, 'Text': 'Ursula K. Le Guin'}], 'Name': 'Ursula K. Le Guin'}, {'Label': 'PERSON', 'Matches': [{'End': 55, 'Start': 44, 'Text': 'Joanna Russ'}], 'Name': 'Joanna Russ'}], 'intent': 'The subject is suggesting that the person should consider reading the works of Ursula K. Le Guin or Joanna Russ.'}}, 'token_count': 18} 0.8534346954749745
```

## Octavia Butler（奥塔维亚·巴特勒）

![Octavia Butler](https://upload.wikimedia.org/wikipedia/commons/6/6d/Octavia_E._Butler.jpg)

Octavia Butler（奥塔维亚·巴特勒）的同时代人包括厄休拉·勒·格恩（Ursula K. Le Guin）、塞缪尔·R·戴拉尼（Samuel R. Delany）和乔安娜·拉斯（Joanna Russ）[20]。

### 奥塔维亚·巴特勒是谁？

奥塔维亚·埃斯特尔·巴特勒（Octavia Estelle Butler）生于1947年6月22日，逝世于2006年2月24日，是一位美国科幻作家[21]。

### 奖项荣誉

奥塔维亚·巴特勒曾获得雨果奖（Hugo Award）、星云奖（Nebula Award）和麦克阿瑟基金会奖（MacArthur Fellowship）[22]。