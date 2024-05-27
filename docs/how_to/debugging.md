# 如何调试你的LLM应用程序
与构建任何类型的软件一样，使用LLM构建时，总会有调试的需求。模型调用可能会失败，模型输出可能格式错误，或者可能存在一些嵌套的模型调用，不清楚在哪一步出现了错误的输出。
有三种主要的调试方法：
- 详细模式：为你的链中的“重要”事件添加打印语句。
- 调试模式：为你的链中的所有事件添加日志记录语句。
- LangSmith跟踪：将事件记录到[LangSmith](https://docs.smith.langchain.com/)，以便在那里进行可视化。
|                        | 详细模式 | 调试模式 | LangSmith跟踪 |
|------------------------|----------|----------|----------------|
| 免费                   | ✅        | ✅      | ✅             |
| 用户界面               | ❌        | ❌      | ✅             |
| 持久化                 | ❌        | ❌      | ✅             |
| 查看所有事件           | ❌        | ✅      | ✅             |
| 查看“重要”事件         | ✅        | ❌      | ✅             |
| 本地运行               | ✅        | ✅      | ❌             |
## 跟踪
使用LangChain构建的许多应用程序将包含多个步骤，其中包含多次LLM调用。
随着这些应用程序变得越来越复杂，能够检查链或代理内部发生了什么变得至关重要。
这样做的最佳方式是使用[LangSmith](https://smith.langchain.com)。
在上面的链接上注册后，请确保设置你的环境变量以开始记录跟踪：
```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```
或者，在笔记本中，你可以使用以下方式设置：
```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
假设我们有一个代理，并且希望可视化它所采取的操作和接收到的工具输出。在没有任何调试的情况下，这是我们看到的：
```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="llm"
/>
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.prompts import ChatPromptTemplate
tools = [TavilySearchResults(max_results=1)]
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant.",
        ),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)
# 构建工具代理
agent = create_tool_calling_agent(llm, tools, prompt)
# 通过传入代理和工具来创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke(
    {"input": "Who directed the 2023 film Oppenheimer and what is their age in days?"}
)
```
```output
{'input': 'Who directed the 2023 film Oppenheimer and what is their age in days?',
 'output': 'The 2023 film "Oppenheimer" was directed by Christopher Nolan.\n\nTo calculate Christopher Nolan\'s age in days, we first need his birthdate, which is July 30, 1970. Let\'s calculate his age in days from his birthdate to today\'s date, December 7, 2023.\n\n1. Calculate the total number of days from July 30, 1970, to December 7, 2023.\n2. Nolan was born on July 30, 1970. From July 30, 1970, to July 30, 2023, is 53 years.\n3. From July 30, 2023, to December 7, 2023, is 130 days.\n\nNow, calculate the total days:\n- 53 years = 53 x 365 = 19,345 days\n- Adding leap years from 1970 to 2023: There are 13 leap years (1972, 1976, 1980, 1984, 1988, 1992, 1996, 2000, 2004, 2008, 2012, 2016, 2020). So, add 13 days.\n- Total days from years and leap years = 19,345 + 13 = 19,358 days\n- Add the days from July 30, 2023, to December 7, 2023 = 130 days\n\nTotal age in days = 19,358 + 130 = 19,488 days\n\nChristopher Nolan is 19,488 days old as of December 7, 2023.'}
```
我们没有得到太多输出，但由于我们设置了LangSmith，我们可以轻松地看到发生了什么：
[https://smith.langchain.com/public/a89ff88f-9ddc-4757-a395-3a1b365655bf/r](https://smith.langchain.com/public/a89ff88f-9ddc-4757-a395-3a1b365655bf/r)
## `set_debug` 和 `set_verbose`
如果你在Jupyter笔记本中进行原型设计或运行Python脚本，打印出链运行的中间步骤可能会有所帮助。
有许多方法可以以不同程度的详细程度启用打印。
注意：即使启用了LangSmith，这些仍然有效，因此你可以同时打开并运行它们。
### `set_verbose(True)`
设置 `verbose` 标志将以稍微更易读的格式打印出输入和输出，并将跳过记录某些原始输出（例如 LLM 调用的令牌使用统计信息），以便您可以专注于应用程序逻辑。
```python
from langchain.globals import set_verbose
set_verbose(True)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke(
    {"input": "Who directed the 2023 film Oppenheimer and what is their age in days?"}
)
```
```output
> 进入新的 AgentExecutor 链...
调用：`tavily_search_results_json`，参数为 `{'query': 'director of the 2023 film Oppenheimer'}`
[{'url': 'https://m.imdb.com/title/tt15398776/', 'content': 'Oppenheimer: Directed by Christopher Nolan. With Cillian Murphy, Emily Blunt, Robert Downey Jr., Alden Ehrenreich. The story of American scientist J. Robert Oppenheimer and his role in the development of the atomic bomb.'}]
调用：`tavily_search_results_json`，参数为 `{'query': 'birth date of Christopher Nolan'}`
[{'url': 'https://m.imdb.com/name/nm0634240/bio/', 'content': 'Christopher Nolan. Writer: Tenet. Best known for his cerebral, often nonlinear, storytelling, acclaimed Academy Award winner writer/director/producer Sir Christopher Nolan CBE was born in London, England. Over the course of more than 25 years of filmmaking, Nolan has gone from low-budget independent films to working on some of the biggest blockbusters ever made and became one of the most ...'}]
调用：`tavily_search_results_json`，参数为 `{'query': 'Christopher Nolan birth date'}`
回复：2023年电影《Oppenheimer》由克里斯托弗·诺兰（Christopher Nolan）执导。
为了计算克里斯托弗·诺兰的年龄（以天为单位），我需要他的确切出生日期。让我为您找到这些信息。
[{'url': 'https://m.imdb.com/name/nm0634240/bio/', 'content': 'Christopher Nolan. Writer: Tenet. Best known for his cerebral, often nonlinear, storytelling, acclaimed Academy Award winner writer/director/producer Sir Christopher Nolan CBE was born in London, England. Over the course of more than 25 years of filmmaking, Nolan has gone from low-budget independent films to working on some of the biggest blockbusters ever made and became one of the most ...'}]
调用：`tavily_search_results_json`，参数为 `{'query': 'Christopher Nolan date of birth'}`
回复：看来我需要细化搜索以获取克里斯托弗·诺兰的确切出生日期。让我再试一次找到这个具体信息。
[{'url': 'https://m.imdb.com/name/nm0634240/bio/', 'content': 'Christopher Nolan. Writer: Tenet. Best known for his cerebral, often nonlinear, storytelling, acclaimed Academy Award winner writer/director/producer Sir Christopher Nolan CBE was born in London, England. Over the course of more than 25 years of filmmaking, Nolan has gone from low-budget independent films to working on some of the biggest blockbusters ever made and became one of the most ...'}]
我目前无法从可用的来源中检索到克里斯托弗·诺兰的确切出生日期。但是，众所周知，他出生于1970年7月30日。使用这个日期，我可以计算他截至今天的年龄（以天为单位）。
让我们计算一下：
- 克里斯托弗·诺兰的出生日期：1970年7月30日。
- 今天的日期：2023年12月7日。
这两个日期之间的天数可以计算如下：
1. 从1970年7月30日到2023年7月30日是53年。
2. 从2023年7月30日到2023年12月7日是130天。
计算53年的总天数（考虑闰年）：
- 53年 × 365天/年 = 19,345天
- 加上闰年（1972年，1976年，...，2020年，2024年 - 13个闰年）：13天
从出生到2023年7月30日的总天数：19,345 + 13 = 19,358天
再加上从2023年7月30日到2023年12月7日的天数：130天
截至2023年12月7日的总年龄（以天为单位）：19,358 + 130 = 19,488天。
因此，截至2023年12月7日，克里斯托弗·诺兰的年龄为19,488天。
> 链结束。
```
```output
{'input': 'Who directed the 2023 film Oppenheimer and what is their age in days?',
 'output': "我目前无法从可用的来源中检索到克里斯托弗·诺兰的确切出生日期。但是，众所周知，他出生于1970年7月30日。使用这个日期，我可以计算他截至今天的年龄（以天为单位）。\n\n让我们计算一下：\n\n- 克里斯托弗·诺兰的出生日期：1970年7月30日。\n- 今天的日期：2023年12月7日。\n\n这两个日期之间的天数可以计算如下：\n\n1. 从1970年7月30日到2023年7月30日是53年。\n2. 从2023年7月30日到2023年12月7日是130天。\n\n计算53年的总天数（考虑闰年）：\n- 53年 × 365天/年 = 19,345天\n- 加上闰年（1972年，1976年，...，2020年，2024年 - 13个闰年）：13天\n\n从出生到2023年7月30日的总天数：19,345 + 13 = 19,358天\n再加上从2023年7月30日到2023年12月7日的天数：130天\n\n截至2023年12月7日的总年龄（以天为单位）：19,358 + 130 = 19,488天。\n\n因此，截至2023年12月7日，克里斯托弗·诺兰的年龄为19,488天。"}
```
### `set_debug(True)`
设置全局的 `debug` 标志将导致所有具有回调支持的 LangChain 组件（链、模型、代理、工具、检索器）打印它们接收的输入和生成的输出。这是最详细的设置，将完全记录原始输入和输出。
```python
from langchain.globals import set_debug
set_debug(True)
set_verbose(False)
agent_executor = AgentExecutor(agent=agent, tools=tools)
agent_executor.invoke(
    {"input": "Who directed the 2023 film Oppenheimer and what is their age in days?"}
)
```
```output
[chain/start] [1:chain:AgentExecutor] 进入带有输入的链运行：
{
  "input": "Who directed the 2023 film Oppenheimer and what is their age in days?"
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence] 进入带有输入的链运行：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad>] 进入带有输入的链运行：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad> > 4:chain:RunnableParallel<agent_scratchpad>] 进入带有输入的链运行：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad> > 4:chain:RunnableParallel<agent_scratchpad> > 5:chain:RunnableLambda] 进入带有输入的链运行：
{
  "input": ""
}
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad> > 4:chain:RunnableParallel<agent_scratchpad> > 5:chain:RunnableLambda] [1ms] 退出链运行，输出：
{
  "output": []
}
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad> > 4:chain:RunnableParallel<agent_scratchpad>] [2ms] 退出链运行，输出：
{
  "agent_scratchpad": []
}
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 3:chain:RunnableAssign<agent_scratchpad>] [5ms] 退出链运行，输出：
{
  "input": "Who directed the 2023 film Oppenheimer and what is their age in days?",
  "intermediate_steps": [],
  "agent_scratchpad": []
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 6:prompt:ChatPromptTemplate] 进入提示运行，输入：
{
  "input": "Who directed the 2023 film Oppenheimer and what is their age in days?",
  "intermediate_steps": [],
  "agent_scratchpad": []
}
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 6:prompt:ChatPromptTemplate] [1ms] 退出提示运行，输出：
[outputs]
[llm/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 7:llm:ChatOpenAI] 进入 LLM 运行，输入：
{
  "prompts": [
    "System: You are a helpful assistant.\nHuman: Who directed the 2023 film Oppenheimer and what is their age in days?"
  ]
}
[llm/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 7:llm:ChatOpenAI] [3.17s] 退出 LLM 运行，输出：
{
  "generations": [
    [
      {
        "text": "",
        "generation_info": {
          "finish_reason": "tool_calls"
        },
        "type": "ChatGenerationChunk",
        "message": {
          "lc": 1,
          "type": "constructor",
          "id": [
            "langchain",
            "schema",
            "messages",
            "AIMessageChunk"
          ],
          "kwargs": {
            "content": "",
            "example": false,
            "additional_kwargs": {
              "tool_calls": [
                {
                  "index": 0,
                  "id": "call_fnfq6GjSQED4iF6lo4rxkUup",
                  "function": {
                    "arguments": "{\"query\": \"director of the 2023 film Oppenheimer\"}",
                    "name": "tavily_search_results_json"
                  },
                  "type": "function"
                },
                {
                  "index": 1,
                  "id": "call_mwhVi6pk49f4OIo5rOWrr4TD",
                  "function": {
                    "arguments": "{\"query\": \"birth date of Christopher Nolan\"}",
                    "name": "tavily_search_results_json"
                  },
                  "type": "function"
                }
              ]
            },
            "tool_call_chunks": [
              {
                "name": "tavily_search_results_json",
                "args": "{\"query\": \"director of the 2023 film Oppenheimer\"}",
                "id": "call_fnfq6GjSQED4iF6lo4rxkUup",
                "index": 0
              },
              {
                "name": "tavily_search_results_json",
                "args": "{\"query\": \"birth date of Christopher Nolan\"}",
                "id": "call_mwhVi6pk49f4OIo5rOWrr4TD",
                "index": 1
              }
            ],
            "response_metadata": {
              "finish_reason": "tool_calls"
            },
```json
{
  "id": "run-6e160323-15f9-491d-aadf-b5d337e9e2a1",
  "tool_calls": [
    {
      "name": "tavily_search_results_json",
      "args": {
        "query": "2023年电影《奥本海默》的导演"
      },
      "id": "call_fnfq6GjSQED4iF6lo4rxkUup"
    },
    {
      "name": "tavily_search_results_json",
      "args": {
        "query": "克里斯托弗·诺兰的出生日期"
      },
      "id": "call_mwhVi6pk49f4OIo5rOWrr4TD"
    }
  ],
  "invalid_tool_calls": []
}
[chain/start] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 8:parser:ToolsAgentOutputParser] 进入解析器运行，输入为：
[inputs]
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence > 8:parser:ToolsAgentOutputParser] [1ms] 解析器运行结束，输出为：
[outputs]
[chain/end] [1:chain:AgentExecutor > 2:chain:RunnableSequence] [3.18s] 链式运行结束，输出为：
[outputs]
[tool/start] [1:chain:AgentExecutor > 9:tool:tavily_search_results_json] 进入工具运行，输入为：
"{'query': '2023年电影《奥本海默》的导演'}"
``````output
在 ConsoleCallbackHandler.on_tool_end 回调中发生错误：AttributeError("'list' object has no attribute 'strip'")
``````output
[tool/start] [1:chain:AgentExecutor > 10:tool:tavily_search_results_json] 进入工具运行，输入为：
"{'query': '克里斯托弗·诺兰的出生日期'}"
``````output
在 ConsoleCallbackHandler.on_tool_end 回调中发生错误：AttributeError("'list' object has no attribute 'strip'")
``````output
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence] 进入链式运行，输入为：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad>] 进入链式运行，输入为：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad> > 13:chain:RunnableParallel<agent_scratchpad>] 进入链式运行，输入为：
{
  "input": ""
}
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad> > 13:chain:RunnableParallel<agent_scratchpad> > 14:chain:RunnableLambda] 进入链式运行，输入为：
{
  "input": ""
}
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad> > 13:chain:RunnableParallel<agent_scratchpad> > 14:chain:RunnableLambda] [1ms] 链式运行结束，输出为：
[outputs]
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad> > 13:chain:RunnableParallel<agent_scratchpad>] [4ms] 链式运行结束，输出为：
[outputs]
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 12:chain:RunnableAssign<agent_scratchpad>] [8ms] 链式运行结束，输出为：
[outputs]
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 15:prompt:ChatPromptTemplate] 进入提示运行，输入为：
[inputs]
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 15:prompt:ChatPromptTemplate] [1ms] 提示运行结束，输出为：
[outputs]
[llm/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 16:llm:ChatOpenAI] 进入LLM运行，输入为：
{
  "prompts": [
系统：你是一个乐于助人的助手。
人类：谁执导了2023年的电影《奥本海默》，他们的年龄是多少天？
AI：
工具：[{"url": "https://m.imdb.com/title/tt15398776/fullcredits/", "content": "《奥本海默》（2023）演职员表，包括演员、导演、编剧等。菜单。...摄影指导：幕后的杰森·加里...最佳男孩抓手...电影装载员卢克·普兰...航空协调员"}]
工具：[{"url": "https://en.wikipedia.org/wiki/Christopher_Nolan", "content": "2003年初，诺兰向华纳兄弟提出了制作一部新的蝙蝠侠电影的想法，基于角色的起源故事。[58] 诺兰对将其扎根于比漫画书幻想更真实的世界的概念深感着迷。[59] 在拍摄期间，他大量依赖传统特技和小型效果，最小程度使用计算机生成的图像（CGI）。[60] 《蝙蝠侠：侠影之谜》（2005）是诺兰迄今为止承担的最大项目，[61] 受到了评论界的好评和商业上的成功。[62][63] 克里斯蒂安·贝尔饰演布鲁斯·韦恩/蝙蝠侠，还有迈克尔·凯恩、加里·奥德曼、摩根·弗里曼和利亚姆·尼森等人出演。《蝙蝠侠：侠影之谜》是2005年票房第九高的电影，因其心理深度和当代相关性而受到赞扬；[63][66] 它被认为是2000年代最具影响力的电影之一。[67] 电影作家伊恩·纳森写道，在他职业生涯的五年内，诺兰“从默默无闻到独立电影的宠儿，再到获得好莱坞最大项目的创意控制权，（或许是不经意间）引发了重新定义整个行业的类型”。[68]\\n诺兰执导、编剧和制作了《致命魔术》（2006），这是根据克里斯托弗·普里斯特的小说改编的，讲述了两位19世纪对手魔术师的故事。[69] 他执导、编剧和剪辑了短片《盗窃》（1996），该片在一个周末用黑白胶片和有限设备以及小型演员和工作人员拍摄。[12][20] 由诺兰资助并使用UCL联合电影协会的设备拍摄，该片于1996年在剑桥电影节上映，被认为是UCL最好的短片之一。[21] 由于未知原因，该电影此后已被从公众视野中移除。[19] 诺兰拍摄了第三部短片《涂鸦虫》（1997），讲述一个男人似乎在用鞋追逐昆虫，却发现那是一个迷你版的自己。[14][22] 诺兰和托马斯在上世纪90年代中期曾试图与拉里·马洪尼合作拍摄一部影片，但他们放弃了。[23] 在他职业生涯的这一时期，诺兰几乎没有成功地推动他的项目，面临着几次拒绝；他补充说，“英国的资金非常有限”。哲学教授大卫·凯尔·约翰逊写道，《《盗梦空间》一上映，几乎立刻就成为了经典》，赞扬了它对哲学思想的探索，包括信仰的飞跃和洞穴寓言。[97] 该电影在全球的票房收入超过了8.36亿美元。[98] 提名了八项奥斯卡奖，包括最佳影片和最佳原创剧本，它赢得了最佳摄影、最佳声音混合、最佳音效剪辑和最佳视觉效果奖。[99] 诺兰获得了英国电影学院奖和金球奖最佳导演提名，以及其他荣誉。[40]\\n在《黑暗骑士崛起》（2012）上映时，英国电影协会的约瑟夫·贝文写了一篇关于他的简介：“在短短十年多的时间里，克里斯托弗·诺兰从有前途的英国独立导演成为了新一代智慧型娱乐的不折不扣的大师。”他进一步写道，诺兰的作品反映了“产品条件的多样性”，从低成本电影到利润丰厚的大片，“广泛的类型和背景”，以及“宣扬他的多才多艺的多样性的风格”。[193] 电影理论家大卫·博德威尔写道，诺兰能够将他的“实验冲动”与主流娱乐的要求融合在一起，描述他的作品为“通过主观视角和交叉剪辑的技术来实验电影时间”。[194] 诺兰对实际、摄影机内效果、小模型的使用以及胶片拍摄的运用在21世纪初的电影中产生了极大的影响。[195][196] IndieWire在2019年写道，诺兰在一个大片制作已经成为“主要由计算机生成的艺术形式”的时代，仍然保持着一个可行的替代模式。[196] 起初不愿意拍续集，但在华纳兄弟多次坚持后，他同意了。[78] 诺兰希望通过扩展画布和采用“城市故事、大型犯罪故事...在这里你会看到警察、司法系统、义务警员、穷人、富人、罪犯”的动态，来扩展第一部电影的黑色质感。[79] 诺兰继续最小化CGI的使用，采用高分辨率IMAX摄像机，使其成为第一部使用这项技术的重要电影。[80][81]"}]
[llm/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 16:llm:ChatOpenAI] [20.22s] LLM 运行结束，输出结果如下：
{
  "generations": [
    [
      {
        "text": "2023年的电影《Oppenheimer》由克里斯托弗·诺兰执导。\n\n要计算克里斯托弗·诺兰的年龄（以天为单位），我们首先需要知道他的出生日期，即1970年7月30日。让我们从他的出生日期到今天的日期，即2023年12月7日，计算他的年龄（以天为单位）。\n\n1. 计算从1970年7月30日到2023年12月7日的总天数。\n2. 克里斯托弗·诺兰出生于1970年7月30日。从1970年7月30日到2023年7月30日，是53年。\n3. 从2023年7月30日到2023年12月7日，是130天。\n\n现在，计算53年的总天数：\n- 每年有365天，所以53年 × 365天/年 = 19,345天。\n- 加上从1970年到2023年的闰年：1972年、1976年、1980年、1984年、1988年、1992年、1996年、2000年、2004年、2008年、2012年、2016年、2020年和2024年（截止到2月）。这给我们14个闰年。\n- 闰年的总天数：14天。\n\n全部加在一起：\n- 总天数 = 19,345天（来自年份）+ 14天（来自闰年）+ 130天（从2023年7月30日到2023年12月7日）= 19,489天。\n\n因此，截至2023年12月7日，克里斯托弗·诺兰已经活了19,489天。",
        "generation_info": {
          "finish_reason": "stop"
        },
        "type": "ChatGenerationChunk",
        "message": {
          "lc": 1,
          "type": "constructor",
          "id": [
            "langchain",
            "schema",
            "messages",
            "AIMessageChunk"
          ],
          "kwargs": {
            "content": "2023年的电影《Oppenheimer》由克里斯托弗·诺兰执导。\n\n要计算克里斯托弗·诺兰的年龄（以天为单位），我们首先需要知道他的出生日期，即1970年7月30日。让我们从他的出生日期到今天的日期，即2023年12月7日，计算他的年龄（以天为单位）。\n\n1. 计算从1970年7月30日到2023年12月7日的总天数。\n2. 克里斯托弗·诺兰出生于1970年7月30日。从1970年7月30日到2023年7月30日，是53年。\n3. 从2023年7月30日到2023年12月7日，是130天。\n\n现在，计算53年的总天数：\n- 每年有365天，所以53年 × 365天/年 = 19,345天。\n- 加上从1970年到2023年的闰年：1972年、1976年、1980年、1984年、1988年、1992年、1996年、2000年、2004年、2008年、2012年、2016年、2020年和2024年（截止到2月）。这给我们14个闰年。\n- 闰年的总天数：14天。\n\n全部加在一起：\n- 总天数 = 19,345天（来自年份）+ 14天（来自闰年）+ 130天（从2023年7月30日到2023年12月7日）= 19,489天。\n\n因此，截至2023年12月7日，克里斯托弗·诺兰已经活了19,489天。",
            "example": false,
            "additional_kwargs": {},
            "tool_call_chunks": [],
            "response_metadata": {
              "finish_reason": "stop"
            },
            "id": "run-1c08a44f-db70-4836-935b-417caaf422a5",
            "tool_calls": [],
            "invalid_tool_calls": []
          }
        }
      }
    ]
  ],
  "llm_output": null,
  "run": null
}
[chain/start] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 17:parser:ToolsAgentOutputParser] 进入解析器运行，输入如下：
[inputs]
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence > 17:parser:ToolsAgentOutputParser] [2ms] 解析器运行结束，输出如下：
[outputs]
[chain/end] [1:chain:AgentExecutor > 11:chain:RunnableSequence] [20.27s] 链式运行结束，输出如下：
[outputs]
[chain/end] [1:chain:AgentExecutor] [26.37s] 链式运行结束，输出如下：
{
  "output": "2023年的电影《Oppenheimer》由克里斯托弗·诺兰执导。\n\n要计算克里斯托弗·诺兰的年龄（以天为单位），我们首先需要知道他的出生日期，即1970年7月30日。让我们从他的出生日期到今天的日期，即2023年12月7日，计算他的年龄（以天为单位）。\n\n1. 计算从1970年7月30日到2023年12月7日的总天数。\n2. 克里斯托弗·诺兰出生于1970年7月30日。从1970年7月30日到2023年7月30日，是53年。\n3. 从2023年7月30日到2023年12月7日，是130天。\n\n现在，计算53年的总天数：\n- 每年有365天，所以53年 × 365天/年 = 19,345天。\n- 加上从1970年到2023年的闰年：1972年、1976年、1980年、1984年、1988年、1992年、1996年、2000年、2004年、2008年、2012年、2016年、2020年和2024年（截止到2月）。这给我们14个闰年。\n- 闰年的总天数：14天。\n\n全部加在一起：\n- 总天数 = 19,345天（来自年份）+ 14天（来自闰年）+ 130天（从2023年7月30日到2023年12月7日）= 19,489天。\n\n因此，截至2023年12月7日，克里斯托弗·诺兰已经活了19,489天。"
}
```
```output
{'input': '谁执导了2023年的电影《Oppenheimer》，他的年龄是多少天？',
2023年的电影《Oppenheimer》由克里斯托弗·诺兰执导。
要计算克里斯托弗·诺兰的年龄（以天为单位），首先需要知道他的出生日期，即1970年7月30日。让我们从他的出生日期到今天的日期，即2023年12月7日，计算他的年龄（以天为单位）。
1. 计算从1970年7月30日到2023年12月7日的总天数。
2. 克里斯托弗·诺兰出生于1970年7月30日。从1970年7月30日到2023年7月30日是53年。
3. 从2023年7月30日到2023年12月7日是130天。
现在，计算53年的总天数：
- 每年有365天，因此53年 × 365天/年 = 19,345天。
- 加上从1970年到2023年的闰年：1972年、1976年、1980年、1984年、1988年、1992年、1996年、2000年、2004年、2008年、2012年、2016年、2020年和2024年（截止到2月）。这给我们14个闰年。
- 闰年的总天数：14天。
将所有天数相加：
- 总天数 = 19,345天（来自年份）+ 14天（来自闰年）+ 130天（从2023年7月30日到2023年12月7日）= 19,489天。
因此，截至2023年12月7日，克里斯托弗·诺兰年龄为19,489天。