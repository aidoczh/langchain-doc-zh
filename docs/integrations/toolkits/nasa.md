# 美国国家航空航天局（NASA）

本文档展示了如何使用代理与 NASA 工具包进行交互。该工具包提供了对 NASA 图像和视频库 API 的访问，未来的版本可能会扩展并包括其他可访问的 NASA API。

**注意：当未指定所需媒体结果的数量时，NASA 图像和视频库的搜索查询可能会导致大量响应。在使用带有 LLM 令牌积分的代理之前，请考虑这一点。**

## 示例用法:

---

### 初始化代理

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.nasa.toolkit import NasaToolkit
from langchain_community.utilities.nasa import NasaAPIWrapper
from langchain_openai import OpenAI
llm = OpenAI(temperature=0, openai_api_key="")
nasa = NasaAPIWrapper()
toolkit = NasaToolkit.from_nasa_api_wrapper(nasa)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

### 查询媒体资产

```python
agent.run(
    "你能找到 2014 年至 2020 年间发布的三张月球图片吗？"
)
```

### 查询媒体资产的详细信息

```python
output = agent.run(
    "我刚刚查询了一张带有 NASA 编号 NHQ_2019_0311_Go Forward to the Moon 的月球图片。"
    "我在哪里可以找到此资产的元数据清单？"
)
```