# Reddit 搜索

在这个笔记本中，我们将学习 Reddit 搜索工具的工作原理。

首先确保你已经使用以下命令安装了 praw：

```python
%pip install --upgrade --quiet  praw
```

然后你需要设置正确的 API 密钥和环境变量。你需要创建一个 Reddit 用户账户并获取凭证。因此，请访问 https://www.reddit.com 并注册一个 Reddit 用户账户。

然后，访问 https://www.reddit.com/prefs/apps 并创建一个应用程序来获取你的凭证。

你应该从创建应用程序中获得你的 client_id 和 secret。现在，你可以将这些字符串粘贴到 client_id 和 client_secret 变量中。

注意：你可以为 user_agent 设置任何字符串。

```python
client_id = ""
client_secret = ""
user_agent = ""
```

```python
from langchain_community.tools.reddit_search.tool import RedditSearchRun
from langchain_community.utilities.reddit_search import RedditSearchAPIWrapper
search = RedditSearchRun(
    api_wrapper=RedditSearchAPIWrapper(
        reddit_client_id=client_id,
        reddit_client_secret=client_secret,
        reddit_user_agent=user_agent,
    )
)
```

然后你可以设置你的查询，例如你想要查询哪个 subreddit，你想要返回多少篇帖子，你想要如何对结果进行排序等。

```python
from langchain_community.tools.reddit_search.tool import RedditSearchSchema
search_params = RedditSearchSchema(
    query="beginner", sort="new", time_filter="week", subreddit="python", limit="2"
)
```

最后运行搜索并获取结果。

```python
result = search.run(tool_input=search_params.dict())
```

```python
print(result)
```

这里是打印结果的一个例子。

注意：你可能会根据 subreddit 中最新的帖子获得不同的输出，但格式应该是类似的。

> 在 r/python 中找到 2 篇帖子：

> 帖子标题：'在 Visual Studio Code 中设置 Github Copilot'

> 用户：Feisty-Recording-715

> Subreddit：r/Python：

> 文本内容：🛠️ 这个教程非常适合想要加强对版本控制理解的初学者，或者对 GitHub 在 Visual Studio Code 中的设置进行快速参考的有经验的开发者。

> 🎓 视频结束时，你将掌握自信地管理你的代码库、与他人合作，并为 GitHub 上的开源项目做出贡献的技能。

> 视频链接：https://youtu.be/IdT1BhrSfdo?si=mV7xVpiyuhlD8Zrw

> 欢迎你的反馈

> 帖子 URL：https://www.reddit.com/r/Python/comments/1823wr7/setup_github_copilot_in_visual_studio_code/

> 帖子类别：N/A.

> 得分：0

> 帖子标题：'使用 pygame 和 PySide6 制作的中国跳棋游戏，支持自定义机器人'

> 用户：HenryChess

> Subreddit：r/Python：

> 文本内容：GitHub 链接：https://github.com/henrychess/pygame-chinese-checkers

> 我不确定这是否算是初学者或中级。我认为我还处于初学者阶段，所以我将其标记为初学者。

> 这是一个供 2 到 3 名玩家玩的中国跳棋（又名 Sternhalma）游戏。我编写的机器人很容易被击败，因为它们主要用于调试代码逻辑部分。但是，你可以编写自己的自定义机器人。在 github 页面上有一个指南。

> 帖子 URL：https://www.reddit.com/r/Python/comments/181xq0u/a_chinese_checkers_game_made_with_pygame_and/

> 帖子类别：N/A.

> 得分：1

## 使用带有代理链的工具

Reddit 搜索功能也作为多输入工具提供。在这个例子中，我们改编了[文档中的现有代码](https://python.langchain.com/v0.1/docs/modules/memory/agent_with_memory/)，并使用 ChatOpenAI 创建了一个带有记忆的代理链。这个代理链能够从 Reddit 获取信息，并使用这些帖子来回应后续的输入。

要运行这个例子，添加你的 Reddit API 访问信息，并从 [OpenAI API](https://help.openai.com/en/articles/4936850-where-do-i-find-my-api-key) 获取 OpenAI 密钥。

```python
# 从 /docs/modules/agents/how_to/sharedmemory_for_tools 中改编的代码
from langchain.agents import AgentExecutor, StructuredChatAgent
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory, ReadOnlySharedMemory
from langchain_community.tools.reddit_search.tool import RedditSearchRun
from langchain_community.utilities.reddit_search import RedditSearchAPIWrapper
from langchain_core.prompts import PromptTemplate
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI
# 提供 Reddit 的密钥
client_id = ""
client_secret = ""
user_agent = ""
# 提供 OpenAI 的密钥
openai_api_key = ""
template = """这是人类和机器人之间的对话：
{chat_history}
为 {input} 写一个对话摘要：
"""
prompt = PromptTemplate(input_variables=["input", "chat_history"], template=template)
memory = ConversationBufferMemory(memory_key="chat_history")
prefix = """与人类进行对话，尽力回答以下问题。你可以使用以下工具："""
suffix = """开始！"""
{chat_history}
问题：{input}
{agent_scratchpad}"""
tools = [
    RedditSearchRun(
        api_wrapper=RedditSearchAPIWrapper(
            reddit_client_id=client_id,
            reddit_client_secret=client_secret,
            reddit_user_agent=user_agent,
        )
    )
]
prompt = StructuredChatAgent.create_prompt(
    prefix=prefix,
    tools=tools,
    suffix=suffix,
    input_variables=["input", "chat_history", "agent_scratchpad"],
)
llm = ChatOpenAI(temperature=0, openai_api_key=openai_api_key)
llm_chain = LLMChain(llm=llm, prompt=prompt)
agent = StructuredChatAgent(llm_chain=llm_chain, verbose=True, tools=tools)
agent_chain = AgentExecutor.from_agent_and_tools(
    agent=agent, verbose=True, memory=memory, tools=tools
)
# 回答第一个提示需要使用 Reddit 搜索工具。
agent_chain.run(input="r/langchain 这周最新的帖子是什么？")
# 回答后续提示使用记忆。
agent_chain.run(input="帖子的作者是谁？")
```

抱歉，我需要您提供需要翻译的英文段落。