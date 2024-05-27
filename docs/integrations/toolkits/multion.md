# MultiOn

[MultiON](https://www.multion.ai/blog/multion-building-a-brighter-future-for-humanity-with-ai-agents) 已经构建了一款可以与各种网络服务和应用程序进行交互的 AI 代理。

这篇笔记本将指导您如何在浏览器中将 LangChain 连接到 `MultiOn` 客户端。

这样可以启用利用 MultiON 代理的自定义代理工作流程。

要使用这个工具包，您需要将 `MultiOn Extension` 添加到您的浏览器中：

- 创建一个 [MultiON 账户](https://app.multion.ai/login?callbackUrl=%2Fprofile)。

- 添加 [Chrome 的 MultiOn 扩展](https://multion.notion.site/Download-MultiOn-ddddcfe719f94ab182107ca2612c07a5)。

```python
%pip install --upgrade --quiet  multion langchain -q
```

```python
from langchain_community.agent_toolkits import MultionToolkit
toolkit = MultionToolkit()
toolkit
```

```output
MultionToolkit()
```

```python
tools = toolkit.get_tools()
tools
```

```output
[MultionCreateSession(), MultionUpdateSession(), MultionCloseSession()]
```

## MultiOn 设置

创建账户后，在 https://app.multion.ai/ 创建一个 API 密钥。

登录以建立与您的扩展程序的连接。

```python
# 授权连接到您的浏览器扩展程序
import multion
multion.login()
```

```output
已登录。
```

## 在代理中使用 Multion 工具包

这将使用 MultiON Chrome 扩展程序执行所需的操作。

我们可以运行下面的代码，并查看 [trace](https://smith.langchain.com/public/34aaf36d-204a-4ce3-a54e-4a0976f09670/r) 来查看：

- 代理使用 `create_multion_session` 工具

- 然后使用 MultiON 执行查询

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
```

```python
# 提示
instructions = """You are an assistant."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```

```python
# LLM
llm = ChatOpenAI(temperature=0)
```

```python
# 代理
agent = create_openai_functions_agent(llm, toolkit.get_tools(), prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=toolkit.get_tools(),
    verbose=False,
)
```

```python
agent_executor.invoke(
    {
        "input": "Use multion to explain how AlphaCodium works, a recently released code language model."
    }
)
```

```output
{'input': 'Use multion to how AlphaCodium works, a recently released code language model.',
 'output': 'AlphaCodium is a recently released code language model that is designed to assist developers in writing code more efficiently. It is based on advanced machine learning techniques and natural language processing. AlphaCodium can understand and generate code in multiple programming languages, making it a versatile tool for developers.\n\nThe model is trained on a large dataset of code snippets and programming examples, allowing it to learn patterns and best practices in coding. It can provide suggestions and auto-complete code based on the context and the desired outcome.\n\nAlphaCodium also has the ability to analyze code and identify potential errors or bugs. It can offer recommendations for improving code quality and performance.\n\nOverall, AlphaCodium aims to enhance the coding experience by providing intelligent assistance and reducing the time and effort required to write high-quality code.\n\nFor more detailed information, you can visit the official AlphaCodium website or refer to the documentation and resources available online.\n\nI hope this helps! Let me know if you have any other questions.'}
```