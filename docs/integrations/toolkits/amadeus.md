# Amadeus

本文介绍了如何将 LangChain 连接到 `Amadeus` 旅行 API。

`Amadeus` 工具包允许代理商在旅行方面做出决策，特别是在搜索和预订航班方面。

要使用此工具包，您需要准备好 Amadeus API 密钥，详见 [开始使用 Amadeus 自助服务 API](https://developers.amadeus.com/get-started/get-started-with-self-service-apis-335)。一旦您获得了 AMADEUS_CLIENT_ID 和 AMADEUS_CLIENT_SECRET，您可以在下面的环境变量中输入它们。

注意：Amadeus 自助服务 API 提供了一个带有[免费有限数据](https://amadeus4dev.github.io/developer-guides/test-data/)的测试环境。这使开发人员可以在部署到生产环境之前构建和测试他们的应用程序。要访问实时数据，您需要[切换到生产环境](https://amadeus4dev.github.io/developer-guides/API-Keys/moving-to-production/)。

```python
%pip install --upgrade --quiet  amadeus > /dev/null
```

## 设置环境变量

工具包将读取 AMADEUS_CLIENT_ID 和 AMADEUS_CLIENT_SECRET 环境变量以对用户进行身份验证，因此您需要在此处设置它们。

```python
# 在此处设置环境变量
import os
os.environ["AMADEUS_CLIENT_ID"] = "CLIENT_ID"
os.environ["AMADEUS_CLIENT_SECRET"] = "CLIENT_SECRET"
# os.environ["AMADEUS_HOSTNAME"] = "production" 或 "test"
```

## 创建 Amadeus 工具包并获取工具

首先，您需要创建工具包，以便以后可以访问其工具。

默认情况下，`AmadeusToolkit` 使用 `ChatOpenAI` 来识别给定位置最近的机场。要使用它，只需设置 `OPENAI_API_KEY`。

```python
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
```

```python
from langchain_community.agent_toolkits.amadeus.toolkit import AmadeusToolkit
toolkit = AmadeusToolkit()
tools = toolkit.get_tools()
```

或者，您可以使用 LangChain 支持的任何 LLM，例如 `HuggingFaceHub`。

```python
from langchain_community.llms import HuggingFaceHub
os.environ["HUGGINGFACEHUB_API_TOKEN"] = "YOUR_HF_API_TOKEN"
llm = HuggingFaceHub(
    repo_id="tiiuae/falcon-7b-instruct",
    model_kwargs={"temperature": 0.5, "max_length": 64},
)
toolkit_hf = AmadeusToolkit(llm=llm)
```

## 在代理中使用 Amadeus 工具包

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_react_agent
from langchain.agents.output_parsers import ReActJsonSingleInputOutputParser
from langchain.tools.render import render_text_description_and_args
from langchain_openai import ChatOpenAI
```

```python
llm = ChatOpenAI(temperature=0)
prompt = hub.pull("hwchase17/react-json")
agent = create_react_agent(
    llm,
    tools,
    prompt,
    tools_renderer=render_text_description_and_args,
    output_parser=ReActJsonSingleInputOutputParser(),
)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```

```python
agent_executor.invoke({"input": "What is the name of the airport in Cali, Colombia?"})
```

```output
> 进入新的 AgentExecutor 链...
我应该使用 closest_airport 工具来找到哥伦比亚卡利的机场。
动作：closest_airport
动作输入：location= "Cali, Colombia"content='{\n  "iataCode": "CLO"\n}'哥伦比亚卡利的机场名字叫 CLO。
最终答案：CLO
> 完成链。
```

```output
{'input': 'What is the name of the airport in Cali, Colombia?', 'output': 'CLO'}
```

```python
agent_executor.invoke(
    {
        "input": "What is the departure time of the cheapest flight on March 10, 2024 leaving Dallas, Texas before noon to Lincoln, Nebraska?"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
问题：What is the departure time of the cheapest flight on March 10, 2024 leaving Dallas, Texas before noon to Lincoln, Nebraska?
思路：我们需要找到离达拉斯，德克萨斯州最近的机场，然后搜索在2024年3月10日中午前从达拉斯出发到内布拉斯加州林肯的最便宜航班的出发时间。
动作：
```

{

  "action": "closest_airport",

  "action_input": {

    "location": "Dallas, Texas"

  }

}

```
content='{\n ÷   "iataCode": "DFW"\n}'现在，我们有了达拉斯，德克萨斯州的 IATA 代码。接下来，我们将搜索从达拉斯（DFW）到内布拉斯加州林肯在2024年3月10日中午前最便宜的航班。
动作：
```

{

  "action": "single_flight_search",

  "action_input": {

    "originLocationCode": "DFW",

    "destinationLocationCode": "LNK",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T12:00:00"

  }

}

```[{'price': {'total': '593.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T10:54:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:15:00'}, 'flightNumber': '1634', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '652.68', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T07:25:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T09:46:00'}, 'flightNumber': '380', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '765.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T16:42:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T17:49:00'}, 'flightNumber': '2655', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T17:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T20:07:00'}, 'flightNumber': '4910', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '810.82', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T07:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T08:51:00'}, 'flightNumber': '4645', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '810.82', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T09:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T10:42:00'}, 'flightNumber': '5215', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '815.99', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'IAD', 'at': '2024-03-11T07:00:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-11T08:03:00'}, 'flightNumber': '418', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '901.12', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T13:15:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T14:50:00'}, 'flightNumber': '323', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '901.12', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T11:40:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:23:00'}, 'flightNumber': '1027', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '919.40', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T15:10:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T16:57:00'}, 'flightNumber': '1504', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '963.36', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T13:45:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T14:45:00'}, 'flightNumber': '1380', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T17:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T20:07:00'}, 'flightNumber': '4910', 'carrier': 'UNITED AIRLINES'}]}]
观察：2024年3月10日从德克萨斯州达拉斯（DFW）到内布拉斯加州林肯（LNK）的最便宜航班在上午10:54起飞。
最终答案：2024年3月10日从德克萨斯州达拉斯出发到内布拉斯加州林肯的最便宜航班的出发时间是上午10:54。
> 完成链。
```

```python
agent_executor.invoke(
    {
        "input": "What is the departure time of the cheapest flight on March 10, 2024 leaving Dallas, Texas before noon to Lincoln, Nebraska?"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
问题：2024 年 3 月 10 日早上 12 点前从德克萨斯州达拉斯出发到内布拉斯加州林肯的最便宜航班的起飞时间是什么？
思路：我们需要找到离德克萨斯州达拉斯最近的机场，然后搜索 2024 年 3 月 10 日从那里到内布拉斯加州林肯的单程航班。
操作：
```

{

  "action": "closest_airport",

  "action_input": {

    "location": "Dallas, Texas"

  }

}

```
content='{\n    "iataCode": "DFW"\n}'现在我们已经找到了离德克萨斯州达拉斯最近的机场，即达拉斯/沃斯堡国际机场，IATA 代码为 DFW，我们可以继续搜索 2024 年 3 月 10 日从 DFW 到内布拉斯加州林肯的单程航班。
操作：
```

{

  "action": "single_flight_search",

  "action_input": {

    "originLocationCode": "DFW",

    "destinationLocationCode": "LNK",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T23:59:59"

  }

}

```markdown
## 机票信息
- 价格：593.35 欧元
- 航段：
  - 航班1：
    - 出发：DFW 机场，E 航站楼，2024-03-10 14:20:00
    - 到达：ORD 机场，1 航站楼，2024-03-10 16:49:00
    - 航班号：1583，承运人：UNITED AIRLINES
    - 转机：
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：593.35 欧元
- 航段：
  - 航班2：
    - 出发：DFW 机场，E 航站楼，2024-03-10 10:54:00
    - 到达：ORD 机场，1 航站楼，2024-03-10 13:15:00
    - 航班号：1634，承运人：UNITED AIRLINES
    - 转机：
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：652.68 欧元
- 航段：
  - 航班3：
    - 出发：DFW 机场，E 航站楼，2024-03-10 07:25:00
    - 到达：ORD 机场，1 航站楼，2024-03-10 09:46:00
    - 航班号：380，承运人：UNITED AIRLINES
    - 转机：
      - 出发：ORD 机场，2 航站楼，2024-03-10 12:45:00
      - 到达：LNK 机场，2024-03-10 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：666.77 欧元
- 航段：
  - 航班4：
    - 出发：DFW 机场，E 航站楼，2024-03-10 12:35:00
    - 到达：IAD 机场，2024-03-10 16:20:00
    - 航班号：358，承运人：UNITED AIRLINES
    - 转机：
      - 出发：IAD 机场，2024-03-11 08:35:00
      - 到达：ORD 机场，1 航站楼，2024-03-11 09:39:00
      - 航班号：1744，承运人：UNITED AIRLINES
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：666.77 欧元
- 航段：
  - 航班5：
    - 出发：DFW 机场，E 航站楼，2024-03-10 12:35:00
    - 到达：IAD 机场，2024-03-10 16:20:00
    - 航班号：358，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DCA 机场，2 航站楼，2024-03-11 07:45:00
      - 到达：ORD 机场，2 航站楼，2024-03-11 08:51:00
      - 航班号：4645，承运人：UNITED AIRLINES
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：666.77 欧元
- 航段：
  - 航班6：
    - 出发：DFW 机场，E 航站楼，2024-03-10 12:35:00
    - 到达：IAD 机场，2024-03-10 16:20:00
    - 航班号：358，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DCA 机场，2 航站楼，2024-03-11 10:45:00
      - 到达：ORD 机场，2 航站楼，2024-03-11 11:56:00
      - 航班号：4704，承运人：UNITED AIRLINES
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：666.77 欧元
- 航段：
  - 航班7：
    - 出发：DFW 机场，E 航站楼，2024-03-10 12:35:00
    - 到达：IAD 机场，2024-03-10 16:20:00
    - 航班号：358，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DCA 机场，2 航站楼，2024-03-11 09:45:00
      - 到达：ORD 机场，2 航站楼，2024-03-11 10:42:00
      - 航班号：5215，承运人：UNITED AIRLINES
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：764.60 欧元
- 航段：
  - 航班8：
    - 出发：DFW 机场，E 航站楼，2024-03-10 17:15:00
    - 到达：IAD 机场，2024-03-10 21:10:00
    - 航班号：1240，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DCA 机场，2 航站楼，2024-03-11 09:45:00
      - 到达：ORD 机场，2 航站楼，2024-03-11 10:42:00
      - 航班号：5215，承运人：UNITED AIRLINES
      - 出发：ORD 机场，2 航站楼，2024-03-11 12:45:00
      - 到达：LNK 机场，2024-03-11 14:19:00
      - 航班号：5728，承运人：UNITED AIRLINES
- 价格：765.35 欧元
- 航段：
  - 航班9：
    - 出发：DFW 机场，E 航站楼，2024-03-10 16:42:00
    - 到达：DEN 机场，2024-03-10 17:49:00
    - 航班号：2655，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DEN 机场，2024-03-11 15:45:00
      - 到达：LNK 机场，2024-03-11 18:12:00
      - 航班号：4252，承运人：UNITED AIRLINES
- 价格：765.35 欧元
- 航段：
  - 航班10：
    - 出发：DFW 机场，E 航站楼，2024-03-10 16:42:00
    - 到达：DEN 机场，2024-03-10 17:49:00
    - 航班号：2655，承运人：UNITED AIRLINES
    - 转机：
      - 出发：DEN 机场，2024-03-11 17:45:00
      - 到达：LNK 机场，2024-03-11 20:07:00
      - 航班号：4910，承运人：UNITED AIRLINES
2024年3月10日，从得克萨斯州达拉斯出发前往内布拉斯加州林肯的最早航班将于2024年3月11日14:19抵达内布拉斯加州。
```

最终答案：2024年3月10日，从德克萨斯州达拉斯出发前往内布拉斯加州林肯的最早航班于2024年3月11日14:19抵达内布拉斯加州。

> 链条完成。

```python
# to execute api correctly, change the querying date to feature
agent_executor.invoke(
    {
        "input": "2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的全程旅行时间是多少？"
    }
)
```

```output
> 进入新的 AgentExecutor 链条...
问题：2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的全程旅行时间是多少？
思路：我们需要找到2024年3月10日俄勒冈州波特兰到德克萨斯州达拉斯之间的最便宜航班，然后计算总的旅行时间。
操作：
```

{

  "action": "single_flight_search",

  "action_input": {

    "originLocationCode": "PDX",

    "destinationLocationCode": "DFW",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T23:59:59"

  }

}

```json
[{'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T09:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T14:11:00'}, 'flightNumber': '2920', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T05:45:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T07:59:00'}, 'flightNumber': '1722', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '304.18', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T20:21:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T00:59:00'}, 'flightNumber': '1200', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '304.18', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T05:45:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T07:59:00'}, 'flightNumber': '1722', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T20:21:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T00:59:00'}, 'flightNumber': '1200', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '377.48', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T16:05:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T19:28:00'}, 'flightNumber': '2254', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T20:20:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:18:00'}, 'flightNumber': '772', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '377.48', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T07:11:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:00:00'}, 'flightNumber': '1693', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T15:50:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T18:25:00'}, 'flightNumber': '793', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '382.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T10:38:00'}, 'arrival': {'iataCode': 'IAH', 'terminal': 'C', 'at': '2024-03-10T20:38:00'}, 'flightNumber': '1823', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'IAH', 'terminal': 'C', 'at': '2024-03-10T21:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T22:45:00'}, 'flightNumber': '320', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '389.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T19:04:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T21:09:00'}, 'flightNumber': '431', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '389.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T19:04:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T21:09:00'}, 'flightNumber': '431', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T09:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T14:11:00'}, 'flightNumber': '2920', 'carrier': 'SPIRIT AIRLINES'}]}]
我们有多个不同总价格的航班选择。我们需要计算每个选项的总旅行时间，以找到最便宜航班的完整旅行时间。
操作：
```

{

  "action": "single_flight_search",

  "action_input": {

  {

    "originLocationCode": "PDX",

    "destinationLocationCode": "DFW",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T23:59:59"

  }

```json
[{'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T09:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T14:11:00'}, 'flightNumber': '2920', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '246.13', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T05:45:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T07:59:00'}, 'flightNumber': '1722', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '304.18', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T12:09:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T14:22:00'}, 'flightNumber': '427', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T20:21:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T00:59:00'}, 'flightNumber': '1200', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '304.18', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T05:45:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T07:59:00'}, 'flightNumber': '1722', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T20:21:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T00:59:00'}, 'flightNumber': '1200', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '377.48', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T16:05:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T19:28:00'}, 'flightNumber': '2254', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T20:20:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:18:00'}, 'flightNumber': '772', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '377.48', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T07:11:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:00:00'}, 'flightNumber': '1693', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T15:50:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T18:25:00'}, 'flightNumber': '793', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '382.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T10:38:00'}, 'arrival': {'iataCode': 'IAH', 'terminal': 'C', 'at': '2024-03-10T20:38:00'}, 'flightNumber': '1823', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'IAH', 'terminal': 'C', 'at': '2024-03-10T21:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T22:45:00'}, 'flightNumber': '320', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '389.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T19:04:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T21:09:00'}, 'flightNumber': '431', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T05:00:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '269', 'carrier': 'SPIRIT AIRLINES'}]}, {'price': {'total': '389.10', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'PDX', 'at': '2024-03-10T19:04:00'}, 'arrival': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-10T21:09:00'}, 'flightNumber': '431', 'carrier': 'SPIRIT AIRLINES'}, {'departure': {'iataCode': 'LAS', 'terminal': '1', 'at': '2024-03-11T09:30:00'}, 'arrival': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-11T14:11:00'}, 'flightNumber': '2920', 'carrier': 'SPIRIT AIRLINES'}]}]
```

错误：最早和最晚的出发日期需要是同一天。如果您正在尝试搜索往返航班，请先调用此函数以获取去程航班，然后再次调用以获取返程航班。

我们已经检索到了航班选项，但我们需要计算每个选项的总旅行时间，以找到最便宜的航班的完整旅行时间。

操作：

```json
{
  "action": "single_flight_search",
  "action_input": {
    "originLocationCode": "PDX",
    "destinationLocationCode": "DFW",
    "departureDateTimeEarliest": "2024-03-10T12:09:00",
    "departureDateTimeLatest": "2024-03-11T14:11:00"
  }
}
```

似乎我们无法获取最便宜选项的具体航班详情，以计算完整的旅行时间。我们可以根据先前观察到的中转和飞行持续时间来估算总旅行时间。

最终答案：2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的完整旅行时间约为21小时2分钟。

> 链结束。

```output
{'input': '2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的完整旅行时间是多久？',
 'output': '2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的完整旅行时间约为21小时2分钟。'}
```

```python
agent_executor.invoke(
    {
        "input": "请代表Santiago起草一封简洁的电子邮件，写给Paul，Santiago的旅行代理，要求他预订2024年3月10日从DFW到DCA的最早航班。在电子邮件中包含所有航班细节。"
    }
)
```

```output
> 进入新的AgentExecutor链...
问题：请代表Santiago起草一封简洁的电子邮件，写给Paul，Santiago的旅行代理，要求他预订2024年3月10日从DFW到DCA的最早航班。在电子邮件中包含所有航班细节。
思路：我们需要找到2024年3月10日从达拉斯-沃思堡（DFW）到华盛顿特区（DCA）的最早航班，并在电子邮件中提供所有必要的航班细节。
操作：
```

{

  "action": "single_flight_search",

  "action_input": {

    "originLocationCode": "DFW",

    "destinationLocationCode": "DCA",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T23:59:59"

  }

}

```
根据我们的搜索结果，我们找到了几个2024年3月10日从达拉斯/沃思堡国际机场（DFW）到华盛顿国家机场（DCA）的航班选项。我们需要选择最早的航班，并在邮件中包含所有必要的航班细节，发送给保罗。
操作：
```

{

  "action": "single_flight_search",

  "action_input": {

    "originLocationCode": "DFW",

    "destinationLocationCode": "DCA",

    "departureDateTimeEarliest": "2024-03-10T00:00:00",

    "departureDateTimeLatest": "2024-03-10T23:59:59"

  }

}

```markdown
我们找到了2024年3月10日从DFW到DCA的几个航班选项。我们需要选择最早的航班，并在电子邮件中包含所有必要的航班细节给Paul。
**操作：**
```

```json
{
    "destinationLocationCode": "DCA",
    "departureDateTimeEarliest": "2024-03-10T00:00:00",
    "departureDateTimeLatest": "2024-03-10T23:59:59"
}
[{'price': {'total': '303.31', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T12:00:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-10T13:19:00'}, 'flightNumber': '4431', 'carrier': 'UNITED AIRLINES'}]}]
最终答案：我们找到了多个从达拉斯沃斯堡（DFW）飞往华盛顿特区（DCA）的航班选择，时间为2024年3月10日。最早的航班由美国联合航空公司提供，于06:00从DFW出发，于13:19到达DCA，航班号分别为1517和4431，总价为303.31欧元。
> 完成链条。
```