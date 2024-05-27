# Ionic购物工具

[Ionic](https://www.ioniccommerce.com/)是一个为AI助手提供即插即用电子商务市场的平台。通过在您的助手中包含[Ionic工具](https://github.com/ioniccommerce/ionic_langchain)，您可以轻松地为用户提供在您的助手内直接购物和交易的能力，并且您将从交易中获得一部分收益。

这是一个基本的Jupyter笔记本，演示了如何将Ionic工具集成到您的助手中。有关如何使用Ionic设置您的助手的更多信息，请参阅Ionic的[文档](https://docs.ioniccommerce.com/introduction)。

这个Jupyter笔记本演示了如何在助手中使用Ionic工具。

**注意：ionic-langchain包由Ionic Commerce团队维护，而不是LangChain的维护者。**

---

## 设置

```python
pip install langchain langchain_openai langchainhub
```

```python
pip install ionic-langchain
```

## 设置助手

```python
from ionic_langchain.tool import Ionic, IonicTool
from langchain import hub
from langchain.agents import AgentExecutor, Tool, create_react_agent
from langchain_openai import OpenAI
# 基于ReAct Agent
# https://python.langchain.com/docs/modules/agents/agent_types/react
# 如需其他助手类型的帮助，请联系support@ionicapi.com。
open_ai_key = "YOUR KEY HERE"
model = "gpt-3.5-turbo-instruct"
temperature = 0.6
llm = OpenAI(openai_api_key=open_ai_key, model_name=model, temperature=temperature)
ionic_tool = IonicTool().tool()
# 该工具附带自己的提示信息，
# 但您也可以通过直接更新description属性来更新它：
ionic_tool.description = str(
    """
Ionic是一款电子商务购物工具。助手使用Ionic Commerce购物工具来查找、发现和比较来自数千家在线零售商的产品。当用户正在寻找产品推荐或试图找到特定产品时，助手应该使用该工具。
用户可以指定要查看结果的数量、最低价格和最高价格。
Ionic工具的输入是一个以逗号分隔的值字符串：
  - 查询字符串（必填，不得包含逗号）
  - 结果数量（默认为4，不超过10）
  - 最低价格（以美分为单位，例如$5变为500）
  - 最高价格（以美分为单位）
例如，如果要寻找5到10美元之间的咖啡豆，工具输入将是`coffee beans, 5, 500, 1000`。
以Markdown格式返回它们，每个工具结果的推荐都要包含完整的PDP URL。例如：
1. 产品1：[价格] -- 链接
2. 产品2：[价格] -- 链接
3. 产品3：[价格] -- 链接
4. 产品4：[价格] -- 链接
"""
)
tools = [ionic_tool]
# create_react_agent的默认提示
prompt = hub.pull("hwchase17/react")
agent = create_react_agent(
    llm,
    tools,
    prompt=prompt,
)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, handle_parsing_errors=True, verbose=True, max_iterations=5
)
```

## 运行

```python
input = (
    "我正在寻找一台新的4K显示器，您能帮我找一些价格低于$1000的选项吗？"
)
agent_executor.invoke({"input": input})
```