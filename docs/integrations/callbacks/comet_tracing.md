# 彗星追踪

有两种方法可以使用 Comet 追踪您的 LangChains 执行过程：

1. 将 `LANGCHAIN_COMET_TRACING` 环境变量设置为 "true"。这是推荐的方法。

2. 手动导入 `CometTracer` 并显式传递它。

```python
import os
import comet_llm
from langchain_openai import OpenAI
os.environ["LANGCHAIN_COMET_TRACING"] = "true"
# 如果未设置 API 密钥，则连接到 Comet
comet_llm.init()
# comet 文档以使用环境变量配置 comet
# https://www.comet.com/docs/v2/api-and-sdk/llm-sdk/configuration/
# 这里我们正在配置 comet 项目
os.environ["COMET_PROJECT_NAME"] = "comet-example-langchain-tracing"
from langchain.agents import AgentType, initialize_agent, load_tools
```

```python
# 使用追踪运行代理。确保设置了 OPENAI_API_KEY 以正确运行此示例。
llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
```

```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run("What is 2 raised to .123243 power?")  # 这应该被追踪
# 类似以下的链的 URL 应该打印在您的控制台上：
# https://www.comet.com/<workspace>/<project_name>
# 可以使用该 URL 查看 Comet 中的 LLM 链。
```

```python
# 现在，我们取消设置环境变量并使用上下文管理器。
if "LANGCHAIN_COMET_TRACING" in os.environ:
    del os.environ["LANGCHAIN_COMET_TRACING"]
from langchain_community.callbacks.tracers.comet import CometTracer
tracer = CometTracer()
# 重新创建 LLM、tools 和 agent，并将回调传递给它们
llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent.run(
    "What is 2 raised to .123243 power?", callbacks=[tracer]
)  # 这应该被追踪
```