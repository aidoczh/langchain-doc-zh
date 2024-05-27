# Streamlit

> **[Streamlit](https://streamlit.io/) 是构建和分享数据应用程序的更快方法。**

> Streamlit 可以在几分钟内将数据脚本转换为可共享的 Web 应用程序。全部使用纯 Python 编写，无需前端经验。

> 在 [streamlit.io/generative-ai](https://streamlit.io/generative-ai) 查看更多示例。

[![在 GitHub Codespaces 中打开](https://github.com/codespaces/badge.svg)](https://codespaces.new/langchain-ai/streamlit-agent?quickstart=1)

在本指南中，我们将演示如何使用 `StreamlitCallbackHandler` 在交互式 Streamlit 应用程序中显示代理的思维和行动。使用 MRKL 代理尝试下面正在运行的应用程序：

<iframe loading="lazy" src="https://langchain-mrkl.streamlit.app/?embed=true&embed_options=light_theme"
    style={{ width: 100 + '%', border: 'none', marginBottom: 1 + 'rem', height: 600 }}
    allow="camera;clipboard-read;clipboard-write;"
></iframe>

## 安装和设置

```bash
pip install langchain streamlit
```

您可以运行 `streamlit hello` 来加载一个示例应用程序并验证安装是否成功。在 Streamlit 的 [入门文档](https://docs.streamlit.io/library/get-started) 中查看完整的说明。

## 显示思维和行动

要创建一个 `StreamlitCallbackHandler`，您只需要提供一个父容器来渲染输出。

```python
from langchain_community.callbacks.streamlit import (
    StreamlitCallbackHandler,
)
import streamlit as st
st_callback = StreamlitCallbackHandler(st.container())
```

可以在 [API 参考](https://api.python.langchain.com/en/latest/callbacks/langchain.callbacks.streamlit.streamlit_callback_handler.StreamlitCallbackHandler.html) 中找到用于自定义显示行为的额外关键字参数。

### 场景 1：使用带有工具的代理

目前主要支持的用例是可视化带有工具的代理的行动（或代理执行器）。您可以在 Streamlit 应用程序中创建一个代理，并简单地将 `StreamlitCallbackHandler` 传递给 `agent.run()`，以便在应用程序中实时可视化思维和行动。

```python
import streamlit as st
from langchain import hub
from langchain.agents import AgentExecutor, create_react_agent, load_tools
from langchain_openai import OpenAI
llm = OpenAI(temperature=0, streaming=True)
tools = load_tools(["ddg-search"])
prompt = hub.pull("hwchase17/react")
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
if prompt := st.chat_input():
    st.chat_message("user").write(prompt)
    with st.chat_message("assistant"):
        st_callback = StreamlitCallbackHandler(st.container())
        response = agent_executor.invoke(
            {"input": prompt}, {"callbacks": [st_callback]}
        )
        st.write(response["output"])
```

**注意：** 您需要设置 `OPENAI_API_KEY` 才能成功运行上述应用程序代码。

最简单的方法是通过 [Streamlit secrets.toml](https://docs.streamlit.io/library/advanced-features/secrets-management) 或其他本地环境管理工具来完成。

### 其他场景

目前 `StreamlitCallbackHandler` 主要用于与 LangChain 代理执行器一起使用。未来将添加对其他代理类型、直接与 Chains 等的支持。

您可能还对使用 [StreamlitChatMessageHistory](/docs/integrations/memory/streamlit_chat_message_history) 与 LangChain 感兴趣。