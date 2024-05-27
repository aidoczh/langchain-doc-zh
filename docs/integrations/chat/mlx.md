# MLX

这篇笔记展示了如何开始使用 `MLX` LLM 作为聊天模型。

具体来说，我们将：

1. 使用 [MLXPipeline](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/llms/mlx_pipeline.py)，

2. 使用 `ChatMLX` 类来使这些 LLM 之一能够与 LangChain 的 [Chat Messages](https://python.langchain.com/docs/modules/model_io/chat/#messages) 抽象进行交互。

3. 演示如何使用开源的 LLM 来驱动一个 `ChatAgent` 流程

```python
%pip install --upgrade --quiet  mlx-lm transformers huggingface_hub
```

## 1. 实例化一个 LLM

有三个 LLM 选项可供选择。

```python
from langchain_community.llms.mlx_pipeline import MLXPipeline
llm = MLXPipeline.from_model_id(
    "mlx-community/quantized-gemma-2b-it",
    pipeline_kwargs={"max_tokens": 10, "temp": 0.1},
)
```

## 2. 实例化 `ChatMLX` 来应用聊天模板

实例化聊天模型并传递一些消息。

```python
from langchain_community.chat_models.mlx import ChatMLX
from langchain_core.messages import HumanMessage
messages = [
    HumanMessage(
        content="当一股不可阻挡的力量遇到一块不可移动的物体会发生什么？"
    ),
]
chat_model = ChatMLX(llm=llm)
```

检查聊天消息如何格式化以进行 LLM 调用。

```python
chat_model._to_chat_prompt(messages)
```

调用模型。

```python
res = chat_model.invoke(messages)
print(res.content)
```

## 3. 将其作为代理进行测试！

在这里，我们将测试 `gemma-2b-it` 作为一个零-shot `ReAct` 代理。下面的示例取自[这里](https://python.langchain.com/docs/modules/agents/agent_types/react#using-chat-models)。

> 注意：要运行本节，您需要将 [SerpAPI Token](https://serpapi.com/) 保存为环境变量：`SERPAPI_API_KEY`

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

配置代理以使用 `react-json` 样式提示以及访问搜索引擎和计算器。

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
        "input": "莱昂纳多·迪卡普里奥的女友是谁？她目前的年龄提高到0.43次方是多少？"
    }
)
```