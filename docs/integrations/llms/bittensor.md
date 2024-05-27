# Bittensor

[Bittensor](https://bittensor.com/) 是一个类似比特币的挖矿网络，其中内置了激励机制，旨在鼓励矿工贡献计算和知识。

`NIBittensorLLM` 是由[Neural Internet](https://neuralinternet.ai/)开发的，由`Bittensor`提供支持。

这个LLM展示了分布式人工智能的真正潜力，通过为您提供来自`Bittensor协议`的最佳响应，其中包括各种人工智能模型，如`OpenAI`，`LLaMA2`等。

用户可以在[验证器端点前端](https://api.neuralinternet.ai/)查看他们的日志、请求和API密钥。但是，目前禁止更改配置；否则，用户的查询将被阻止。

如果您遇到任何困难或有任何问题，请随时在[GitHub](https://github.com/Kunj-2206)上联系我们的开发人员，或加入我们的Discord服务器获取最新更新和查询[Neural Internet](https://discord.gg/neuralinternet)。

## NIBittensorLLM 的不同参数和响应处理

```python
import json
from pprint import pprint
from langchain.globals import set_debug
from langchain_community.llms import NIBittensorLLM
set_debug(True)
# NIBittensorLLM中的系统参数是可选的，但您可以设置任何您想要执行的模型任务
llm_sys = NIBittensorLLM(
    system_prompt="您的任务是根据用户提示确定响应。像我解释一下，我是一个项目的技术负责人"
)
sys_resp = llm_sys(
    "什么是bittensor，分布式人工智能的潜在好处是什么？"
)
print(f"设置系统提示后LLM提供的响应是：{sys_resp}")
# top_responses参数可以基于其参数值给出多个响应
# 以下代码检索前10个矿工的响应，所有响应都以json格式呈现
# Json响应结构是
""" {
    "choices":  [
                    {"index": Bittensor的Metagraph索引号,
                    "uid": 矿工的唯一标识符,
                    "responder_hotkey": 矿工的热键,
                    "message":{"role":"assistant","content": 包含实际响应的内容},
                    "response_ms": 从矿工获取响应所需的毫秒数} 
                ]
    } """
multi_response_llm = NIBittensorLLM(top_responses=10)
multi_resp = multi_response_llm.invoke("什么是神经网络的馈送机制？")
json_multi_resp = json.loads(multi_resp)
pprint(json_multi_resp)
```

## 使用LLMChain和PromptTemplate与NIBittensorLLM

```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import NIBittensorLLM
from langchain_core.prompts import PromptTemplate
set_debug(True)
template = """问题：{question}
答案：让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
# NIBittensorLLM中的系统参数是可选的，但您可以设置任何您想要执行的模型任务
llm = NIBittensorLLM(
    system_prompt="您的任务是根据用户提示确定响应。"
)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "什么是bittensor？"
llm_chain.run(question)
```

## 使用Conversational Agent和Google Search Tool与NIBittensorLLM

```python
from langchain_community.utilities import GoogleSearchAPIWrapper
from langchain_core.tools import Tool
search = GoogleSearchAPIWrapper()
tool = Tool(
    name="Google Search",
    description="搜索Google获取最新结果。",
    func=search.run,
)
```

```python
from langchain import hub
from langchain.agents import (
    AgentExecutor,
    create_react_agent,
)
from langchain.memory import ConversationBufferMemory
from langchain_community.llms import NIBittensorLLM
tools = [tool]
prompt = hub.pull("hwchase17/react")
llm = NIBittensorLLM(
    system_prompt="您的任务是根据用户提示确定响应。"
)
memory = ConversationBufferMemory(memory_key="chat_history")
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, memory=memory)
response = agent_executor.invoke({"input": prompt})
```