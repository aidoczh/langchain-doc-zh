# Hugging Face
这篇笔记展示了如何开始使用 `Hugging Face` 的 LLM 作为聊天模型。
具体来说，我们将：
1. 利用 [HuggingFaceEndpoint](https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/llms/huggingface_endpoint.py) 集成来实例化一个 `LLM`。
2. 利用 `ChatHuggingFace` 类来使这些 LLM 中的任何一个与 LangChain 的 [Chat Messages](/docs/concepts/#message-types) 抽象进行交互。
3. 探索使用 `ChatHuggingFace` 进行工具调用。
4. 演示如何使用开源的 LLM 来驱动一个 `ChatAgent` 流水线。
> 注意：要开始使用，您需要将 [Hugging Face Access Token](https://huggingface.co/docs/hub/security-tokens) 保存为环境变量：`HUGGINGFACEHUB_API_TOKEN`。
```python
%pip install --upgrade --quiet  langchain-huggingface text-generation transformers google-search-results numexpr langchainhub sentencepiece jinja2
```
## 1. 实例化一个 LLM
### `HuggingFaceEndpoint`
```python
from langchain_huggingface import HuggingFaceEndpoint
llm = HuggingFaceEndpoint(
    repo_id="meta-llama/Meta-Llama-3-70B-Instruct",
    task="text-generation",
    max_new_tokens=512,
    do_sample=False,
    repetition_penalty=1.03,
)
```
## 2. 实例化 `ChatHuggingFace` 以应用聊天模板
实例化聊天模型并传递一些消息。
**注意**：如果使用自托管的 `text-generation-inference`，则需要显式传递 `model_id`。
```python
from langchain_core.messages import (
    HumanMessage,
    SystemMessage,
)
from langchain_huggingface import ChatHuggingFace
messages = [
    SystemMessage(content="You're a helpful assistant"),
    HumanMessage(
        content="What happens when an unstoppable force meets an immovable object?"
    ),
]
chat_model = ChatHuggingFace(llm=llm)
```
```output
在词汇表中添加了特殊标记，请确保相关的词嵌入已经进行了微调或训练。
```
检查 `model_id`
```python
chat_model.model_id
```
```output
'meta-llama/Meta-Llama-3-70B-Instruct'
```
检查聊天消息格式化为 LLM 调用的方式
```python
chat_model._to_chat_prompt(messages)
```
```output
"<|begin_of_text|><|start_header_id|>system<|end_header_id|>\n\nYou're a helpful assistant<|eot_id|><|start_header_id|>user<|end_header_id|>\n\nWhat happens when an unstoppable force meets an immovable object?<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n"
```
调用模型
```python
res = chat_model.invoke(messages)
print(res.content)
```
```output
在物理学中的一个经典思想实验！
不可阻挡的力量遇到不可移动的物体的概念是一个悖论，几个世纪以来一直困扰着哲学家和物理学家。这是一个扭曲思维的场景，挑战着我们对基本物理定律的理解。
实质上，不可阻挡的力量是无法停止或减速的东西，而不可移动的物体是无法移动或位移的东西。如果我们假设这两个实体存在于同一个宇宙中，我们就会遇到一个逻辑矛盾。
```
## 3. 使用 `ChatHuggingFace` 探索工具调用
`text-generation-inference` 支持从 v2.0.1 开始使用开源 LLM 进行工具调用。
创建一个基本工具（`Calculator`）：
```python
from langchain_core.pydantic_v1 import BaseModel, Field
class Calculator(BaseModel):
    """将两个整数相乘。"""
    a: int = Field(..., description="第一个整数")
    b: int = Field(..., description="第二个整数")
```
将工具绑定到 `chat_model` 并尝试一下：
```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
llm_with_multiply = chat_model.bind_tools([Calculator], tool_choice="auto")
parser = PydanticToolsParser(tools=[Calculator])
tool_chain = llm_with_multiply | parser
tool_chain.invoke("How much is 3 multiplied by 12?")
```
```output
[Calculator(a=3, b=12)]
```
## 4. 作为代理进行测试！
在这里，我们将测试 `Zephyr-7B-beta` 作为一个零-shot `ReAct` 代理。下面的示例摘自[这里](https://python.langchain.com/v0.1/docs/modules/agents/agent_types/react/#using-chat-models)。
> 注意：要运行这一部分，您需要将 [SerpAPI Token](https://serpapi.com/) 保存为环境变量：`SERPAPI_API_KEY`。
```python
from langchain import hub
from langchain.agents import AgentExecutor, load_tools
from langchain.agents.format_scratchpad import format_log_to_str
from langchain.agents.output_parsers import (
    ReActJsonSingleInputOutputParser,
)
from langchain.tools.render import render_text_description
from langchain_community.utilities import SerpAPIWrapper
```
配置代理，使用 `react-json` 样式提示和访问搜索引擎和计算器。
```python
# 设置工具
tools = load_tools(["serpapi", "llm-math"], llm=llm)
# 设置 ReAct 样式提示
prompt = hub.pull("hwchase17/react-json")
prompt = prompt.partial(
    tools=render_text_description(tools),
    tool_names=", ".join([t.name for t in tools]),
)
# 定义代理
chat_model_with_stop = chat_model.bind(stop=["\nObservation"])
agent = (
    {
        "input": lambda x: x["input"],
        "agent_scratchpad": lambda x: format_log_to_str(x["intermediate_steps"]),
    }
    | prompt
    | chat_model_with_stop
    | ReActJsonSingleInputOutputParser()
)
# 实例化 AgentExecutor
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```
```python
agent_executor.invoke(
    {
        "input": "莱昂纳多·迪卡普里奥的女朋友是谁？她的当前年龄提高到0.43次方是多少？"
    }
)
```
```output
> 进入新的 AgentExecutor 链...
问题：莱昂纳多·迪卡普里奥的女朋友是谁？她的当前年龄提高到0.43次方是多少？
思考：我需要使用搜索工具找出莱昂纳多·迪卡普里奥目前的女朋友是谁。然后，我可以使用计算器工具将她的当前年龄提高到0.43次方。
行动：
```
{
  "action": "Search",
  "action_input": "leo dicaprio girlfriend"
}
```
莱昂纳多·迪卡普里奥可能已经在维多利亚·塞雷蒂（Vittoria Ceretti）身上找到了真爱。一位消息人士在《美国周刊》最新一期中独家透露：“他们相爱了。莱昂纳多显然非常自豪地展示维多利亚，并让每个人都看到他们在一起是多么幸福。”现在我们知道莱昂纳多·迪卡普里奥目前的女朋友是维多利亚·塞雷蒂，让我们来了解一下她的当前年龄。
行动：
```
{
  "action": "Search",
  "action_input": "vittoria ceretti age"
}
```
25岁现在我们知道维多利亚·塞雷蒂的当前年龄是25岁，让我们使用计算器工具将其提高到0.43次方。
行动：
```
{
  "action": "Calculator",
  "action_input": "25^0.43"
}
```
答案：3.991298452658078最终答案：维多利亚·塞雷蒂，莱昂纳多·迪卡普里奥的当前女朋友，当提高到0.43次方时，约为4.0，保留两位小数。她的当前年龄为25岁。
> 链结束。
```
```output
{'input': "莱昂纳多·迪卡普里奥的女朋友是谁？她的当前年龄提高到0.43次方是多少？",
 'output': "维多利亚·塞雷蒂，莱昂纳多·迪卡普里奥的当前女朋友，当提高到0.43次方时，约为4.0，保留两位小数。她的当前年龄为25岁。"}
```
哇喔！我们的开源7b参数 Zephyr 模型能够：
1. 规划一系列行动：“我需要使用搜索工具找出莱昂纳多·迪卡普里奥目前的女朋友是谁。然后，我可以使用计算器工具将她的当前年龄提高到0.43次方。”
2. 然后使用 SerpAPI 工具执行搜索，找出莱昂纳多·迪卡普里奥目前的女朋友是谁
3. 执行另一个搜索，找出她的年龄
4. 最后使用计算器工具计算她的年龄提高到0.43次方
看到开源 LLM 作为通用推理代理的潜力是令人兴奋的。你也来试试吧！
```