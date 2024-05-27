# 如何跟踪LLM的令牌使用情况

本笔记将介绍如何跟踪特定调用的令牌使用情况。目前，这仅针对OpenAI API实现。

让我们首先看一个非常简单的示例，用于跟踪单个LLM调用的令牌使用情况。

```python
from langchain_community.callbacks import get_openai_callback
from langchain_openai import OpenAI
```

```python
llm = OpenAI(model_name="gpt-3.5-turbo-instruct", n=2, best_of=2)
```

```python
with get_openai_callback() as cb:
    result = llm.invoke("Tell me a joke")
    print(cb)
```

```output
使用的令牌数：37
	提示令牌：4
	完成令牌：33
成功的请求：1
总成本（美元）：$7.2e-05
```

上下文管理器内的任何内容都将被跟踪。以下是使用它来跟踪连续多次调用的示例。

```python
with get_openai_callback() as cb:
    result = llm.invoke("Tell me a joke")
    result2 = llm.invoke("Tell me a joke")
    print(cb.total_tokens)
```

```output
72
```

如果使用具有多个步骤的链或代理，它将跟踪所有这些步骤。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
tools = load_tools(["serpapi", "llm-math"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
with get_openai_callback() as cb:
    response = agent.run(
        "Who is Olivia Wilde's boyfriend? What is his current age raised to the 0.23 power?"
    )
    print(f"总令牌数：{cb.total_tokens}")
    print(f"提示令牌：{cb.prompt_tokens}")
    print(f"完成令牌：{cb.completion_tokens}")
    print(f"总成本（美元）：${cb.total_cost}")
```

```output
> 进入新的AgentExecutor链...
 我需要找出奥利维亚·怀尔德的男朋友是谁，然后计算他目前的年龄提高到0.23次方。
动作：搜索
动作输入：“奥利维亚·怀尔德 男朋友”
观察：["奥利维亚·怀尔德和哈里·斯泰尔斯的轰动浪漫开始于他们在《别担心，亲爱的》的片场相遇时。", '奥利维亚·怀尔德在结束了与杰森·苏戴奇长达多年的订婚后开始约会哈里·斯泰尔斯——看看他们的恋情时间线。', '奥利维亚·怀尔德和哈里·斯泰尔斯在他们的恋情早期被发现在伦敦四处走动。(. 图片 ...', "看起来奥利维亚·怀尔德和杰森·苏戴奇斯在2023年的开始过得很好。在他们备受瞩目的监护权斗争中——以及女演员的 ...", '两人在2020年分手后开始约会。然而，他们的关系在去年11月结束了。', "奥利维亚·怀尔德和哈里·斯泰尔斯在拍摄《别担心，亲爱的》期间开始约会。虽然这部电影因为 ...", "到目前为止，我们对哈里·斯泰尔斯和奥利维亚·怀尔德的关系了解多少。", '奥利维亚和格莱美奖得主在她与前未婚夫分手仅两个月后开始恋情，他们的恋情始于 ...', "哈里·斯泰尔斯和奥利维亚·怀尔德首次在《别担心，亲爱的》片场相遇，并于2021年1月作为一对出现。重温他们最大的关系 ..."]
思考：哈里·斯泰尔斯是奥利维亚·怀尔德的男朋友。
动作：搜索
动作输入：“哈里·斯泰尔斯 年龄”
观察：29岁
思考：我需要计算29的0.23次方。
动作：计算器
动作输入：29^0.23
观察：答案：2.169459462491557
思考：我现在知道最终答案。
最终答案：哈里·斯泰尔斯是奥利维亚·怀尔德的男朋友，他目前的年龄提高到0.23次方为2.169459462491557。
> 完成链。
总令牌数：2205
提示令牌：2053
完成令牌：152
总成本（美元）：$0.0441
```