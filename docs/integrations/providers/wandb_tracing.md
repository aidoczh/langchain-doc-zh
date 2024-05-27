# WandB 追踪

有两种推荐的方法可以追踪你的 LangChains：

1. 将 `LANGCHAIN_WANDB_TRACING` 环境变量设置为 "true"。

1. 使用一个上下文管理器，通过 `tracing_enabled()` 来追踪特定的代码块。

**注意** 如果环境变量被设置，所有的代码都将被追踪，不管它是否在上下文管理器内部。

```python
import os
from langchain_community.callbacks import wandb_tracing_enabled
os.environ["LANGCHAIN_WANDB_TRACING"] = "true"
# wandb 文档配置 wandb 使用环境变量
# https://docs.wandb.ai/guides/track/advanced/environment-variables
# 这里我们配置 wandb 项目名称
os.environ["WANDB_PROJECT"] = "langchain-tracing"
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```

```python
# 使用追踪运行代理。确保设置 OPENAI_API_KEY 以正确运行此示例。
llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run("2 的 .123243 次方是多少？")  # 这应该被追踪
# 一个类似以下的追踪会在你的控制台中打印出来：
# https://wandb.ai/<wandb_entity>/<wandb_project>/runs/<run_id>
# 这个链接可以用来在 wandb 中查看追踪会话。
```

```python
# 现在，我们取消环境变量的设置，并使用一个上下文管理器。
if "LANGCHAIN_WANDB_TRACING" in os.environ:
    del os.environ["LANGCHAIN_WANDB_TRACING"]
# 使用上下文管理器启用追踪
with wandb_tracing_enabled():
    agent.run("5 的 .123243 次方是多少？")  # 这应该被追踪
agent.run("2 的 .123243 次方是多少？")  # 这不应该被追踪
```

```output
> 进入新的 AgentExecutor 链...
 我需要使用计算器来解决这个问题。
动作：计算器
动作输入：5^.123243
观察：答案：1.2193914912400514
思考：我现在知道最终答案了。
最终答案：1.2193914912400514
> 链结束。
```

```output
> 进入新的 AgentExecutor 链...
 我需要使用计算器来解决这个问题。
动作：计算器
动作输入：2^.123243
观察：答案：1.0891804557407723
思考：我现在知道最终答案了。
最终答案：1.0891804557407723
> 链结束。
```

```output
'1.0891804557407723'
```