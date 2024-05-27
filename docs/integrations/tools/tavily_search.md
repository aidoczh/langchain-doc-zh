# Tavily 搜索

[Tavily 的搜索 API](https://tavily.com) 是一个专门为 AI 代理人（LLMs）构建的搜索引擎，能够以实时、准确和事实为基础的速度提供结果。

## 设置

集成位于 `langchain-community` 包中。我们还需要安装 `tavily-python` 包本身。

```bash
pip install -U langchain-community tavily-python
```

我们还需要设置 Tavily API 密钥。

```python
import getpass
import os
os.environ["TAVILY_API_KEY"] = getpass.getpass()
```

设置 [LangSmith](https://smith.langchain.com/) 也是有帮助的（但不是必须的），以获得最佳的可观察性。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 使用

这里我们展示如何单独使用该工具。

```python
from langchain_community.tools.tavily_search import TavilySearchResults
tool = TavilySearchResults()
```
```python
tool.invoke({"query": "最新的 Burning Man 洪水中发生了什么"})
```
```output
[{'url': 'https://apnews.com/article/burning-man-flooding-nevada-stranded-3971a523f4b993f8f35e158fd1a17a1e',
  'content': 'festival goers are helped off a truck from the Burning Man festival site in Black Rock, Nev., on Monday, Sept. 4, 2023.  festival goers are helped off a truck from the Burning Man festival site in Black Rock, Nev., on Monday, Sept. 4, 2023.  at the site of the Burning Man festival where thousands of attendees remained stranded as flooding from storms swept  at the site of the Burning Man festival where thousands of attendees remained stranded as flooding from storms sweptRENO, Nev. (AP) — The traffic jam leaving the Burning Man festival eased up considerably Tuesday as the exodus from the mud-caked Nevada desert entered another day following massive rain that left tens of thousands of partygoers stranded for days.'},
 {'url': 'https://www.theguardian.com/culture/2023/sep/03/burning-man-nevada-festival-floods',
  'content': 'Officials investigate death at Burning Man as thousands stranded by floods  Burning Man festival-goers trapped in desert as rain turns site to mud  the burning of a giant sculpture to cap off the event, if weather permits. The festival said the roads remain too wet  Burning Man festivalgoers surrounded by mud in Nevada desert – videoMichael Sainato @msainat1 Sun 3 Sep 2023 14.31 EDT Over 70,000 attendees of the annual Burning Man festival in the Black Rock desert of Nevada are stranded as the festival comes to a close on...'},
 {'url': 'https://abcnews.go.com/US/burning-man-flooding-happened-stranded-festivalgoers/story?id=102908331',
  'content': 'ABC News Video Live Shows Election 2024 538 Stream on Burning Man flooding: What happened to stranded festivalgoers?  Tens of thousands of Burning Man attendees are now able to leave the festival after a downpour and massive flooding  Burning Man has been hosted for over 30 years, according to a statement from the organizers.  people last year, and just as many were expected this year. Burning Man began on Aug. 28 and was scheduled to runJulie Jammot/AFP via Getty Images Tens of thousands of Burning Man attendees are now able to leave the festival after a downpour and massive flooding left them stranded over the weekend. The festival, held in the Black Rock Desert in northwestern Nevada, was attended by more than 70,000 people last year, and just as many were expected this year.'},
 {'url': 'https://www.vox.com/culture/2023/9/6/23861675/burning-man-2023-mud-stranded-climate-change-playa-foot',
  'content': 'This year’s rains opened the floodgates for Burning Man criticism  Pray for him people #burningman #burningman2023 #titanicsound #mud #festival  who went to Burning Man  that large wooden Man won’t be the only one burning.Celebrity Culture The Burning Man flameout, explained Climate change — and schadenfreude — finally caught up to the survivalist cosplayers. By Aja Romano @ajaromano Sep 6, 2023, 3:00pm EDT Share'},
 {'url': 'https://www.cnn.com/2023/09/03/us/burning-man-storms-shelter-sunday/index.html',
  'content': 'Editor’s Note: Find the latest Burning Man festival coverage here.  CNN values your feedback More than 70,000 Burning Man festival attendees remain stuck in Nevada desert after rain  Burning Man organizers said Sunday night.  Thousands of people remain trapped at the Burning Man festival in the Nevada desert after heavy rains inundated the"A mucky, muddy, environment." This is what Burning Man looks like See More Videos Editor\'s Note: Find the latest Burning Man festival coverage here. CNN —'}]
```

## 链接

我们在这里展示如何将其作为 [代理人](/docs/tutorials/agents) 的一部分使用。我们使用 OpenAI Functions 代理人，因此我们需要为其设置和安装所需的依赖项。我们还将使用 [LangSmith Hub](https://smith.langchain.com/hub) 来获取提示，因此我们需要安装它。

```bash
pip install -U langchain-openai langchainhub
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```
```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
instructions = """你是一个助手。"""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
llm = ChatOpenAI(temperature=0)
tavily_tool = TavilySearchResults()
tools = [tavily_tool]
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```
```python
agent_executor.invoke({"input": "最新的 Burning Man 洪水事件是什么情况？"})
```
```output

> 进入新的 AgentExecutor 链...

调用: `tavily_search_results_json`，参数为 `{'query': 'latest burning man floods'}`

```
最新的 Burning Man 节日遭遇了大雨，导致发生了洪水和泥泞的情况。成千上万的参与者被困在内华达州的节日现场。有人虚假声称发生了埃博拉疫情和国家紧急状态，但并未宣布紧急状态。据报道，节日现场发生了一起死亡事件，目前正在接受调查。尽管条件艰难，许多节日参与者仍然毫不在意，并继续享受活动。随着被泥浆覆盖的沙漠开始干涸，节日现场的撤离行动开始进行。当局发布了一项山洪暴发警报，并正在对节日中的死亡事件展开调查。
> 完成链。
```

最新的 Burning Man 节庆经历了大雨，导致发生了洪水和泥泞的情况。成千上万的参与者被困在内华达州的节庆现场。有虚假声称爆发了埃博拉疫情和国家紧急情况，但并未宣布紧急状态。据报道节庆现场有一人死亡，目前正在调查中。尽管条件艰难，许多节庆参与者仍然毫不动摇地继续享受活动。随着被泥浆覆盖的沙漠开始干涸，节庆现场的撤离工作已经开始。当局发布了山洪暴发警报，有关节庆死亡事件的调查仍在进行中。[20]

![Burning Man](https://example.com/burning_man_image.jpg)