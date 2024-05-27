# Shell（bash）

在一个沙盒环境之外，给予代理访问 shell 是强大的（尽管有风险）。

LLM 可以使用它来执行任何 shell 命令。一个常见的用例是让 LLM 与您的本地文件系统交互。

**注意：**Shell 工具在 Windows 操作系统上不可用。

```python
from langchain_community.tools import ShellTool
shell_tool = ShellTool()
```

```python
print(shell_tool.run({"commands": ["echo 'Hello World!'", "time"]}))
```

```output
Hello World!
real	0m0.000s
user	0m0.000s
sys	0m0.000s
```

```output
/Users/wfh/code/lc/lckg/langchain/tools/shell/tool.py:34: UserWarning: The shell tool has no safeguards by default. Use at your own risk.
  warnings.warn(
```

### 与代理一起使用

与所有工具一样，可以将它们提供给代理以完成更复杂的任务。让代理从网页中获取一些链接。

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0)
shell_tool.description = shell_tool.description + f"args {shell_tool.args}".replace(
    "{", "{{"
).replace("}", "}}")
self_ask_with_search = initialize_agent(
    [shell_tool], llm, agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
self_ask_with_search.run(
    "Download the langchain.com webpage and grep for all urls. Return only a sorted list of them. Be sure to use double quotes."
)
```

```output
> 进入新的 AgentExecutor 链...
问题：任务是什么？
思考：我们需要下载 langchain.com 网页并从中提取所有的 URL。然后我们需要对这些 URL 进行排序并返回它们。
操作：
```

{

  "action": "shell",

  "action_input": {

    "commands": [

      "curl -s https://langchain.com | grep -o 'http[s]*://[^\" ]*' | sort"

    ]

  }

}

```output
/Users/wfh/code/lc/lckg/langchain/tools/shell/tool.py:34: UserWarning: The shell tool has no safeguards by default. Use at your own risk.
  warnings.warn(
```

```output
Observation: https://blog.langchain.dev/
https://discord.gg/6adMQxSpJS
https://docs.langchain.com/docs/
https://github.com/hwchase17/chat-langchain
https://github.com/hwchase17/langchain
https://github.com/hwchase17/langchainjs
https://github.com/sullivan-sean/chat-langchainjs
https://js.langchain.com/docs/
https://python.langchain.com/en/latest/
https://twitter.com/langchainai
Thought: URL 已成功提取并排序。我们可以将 URL 列表作为最终答案返回。
最终答案：["https://blog.langchain.dev/", "https://discord.gg/6adMQxSpJS", "https://docs.langchain.com/docs/", "https://github.com/hwchase17/chat-langchain", "https://github.com/hwchase17/langchain", "https://github.com/hwchase17/langchainjs", "https://github.com/sullivan-sean/chat-langchainjs", "https://js.langchain.com/docs/", "https://python.langchain.com/en/latest/", "https://twitter.com/langchainai"]
> 链结束。
```

```output
'["https://blog.langchain.dev/", "https://discord.gg/6adMQxSpJS", "https://docs.langchain.com/docs/", "https://github.com/hwchase17/chat-langchain", "https://github.com/hwchase17/langchain", "https://github.com/hwchase17/langchainjs", "https://github.com/sullivan-sean/chat-langchainjs", "https://js.langchain.com/docs/", "https://python.langchain.com/en/latest/", "https://twitter.com/langchainai"]'
```