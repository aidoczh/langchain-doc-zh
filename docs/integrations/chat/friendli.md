# ChatFriendli

> [Friendli](https://friendli.ai/) 通过可扩展、高效的部署选项，为高需求的 AI 工作负载量身定制，提高了 AI 应用的性能并优化了成本节约。

本教程将指导您如何使用 LangChain 集成 `ChatFriendli` 到聊天应用程序中。`ChatFriendli` 提供了一种灵活的方法来生成对话式 AI 响应，支持同步和异步调用。

## 设置

确保已安装 `langchain_community` 和 `friendli-client`。

```sh
pip install -U langchain-comminity friendli-client.
```

登录 [Friendli Suite](https://suite.friendli.ai/) 创建个人访问令牌，并将其设置为 `FRIENDLI_TOKEN` 环境变量。

```python
import getpass
import os
os.environ["FRIENDLI_TOKEN"] = getpass.getpass("Friendi Personal Access Token: ")
```

您可以通过选择要使用的模型来初始化 Friendli 聊天模型。默认模型是 `mixtral-8x7b-instruct-v0-1`。您可以在 [docs.friendli.ai](https://docs.periflow.ai/guides/serverless_endpoints/pricing#text-generation-models) 上查看可用的模型。

```python
from langchain_community.chat_models.friendli import ChatFriendli
chat = ChatFriendli(model="llama-2-13b-chat", max_tokens=100, temperature=0)
```

## 使用

`FrienliChat` 支持 [`ChatModel`](/docs/how_to#chat-models) 的所有方法，包括异步 API。

您还可以使用 `invoke`、`batch`、`generate` 和 `stream` 的功能。

```python
from langchain_core.messages.human import HumanMessage
from langchain_core.messages.system import SystemMessage
system_message = SystemMessage(content="Answer questions as short as you can.")
human_message = HumanMessage(content="Tell me a joke.")
messages = [system_message, human_message]
chat.invoke(messages)
```

```output
AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!")
```

```python
chat.batch([messages, messages])
```

```output
[AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"),
 AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!")]
```

```python
chat.generate([messages, messages])
```

```output
LLMResult(generations=[[ChatGeneration(text=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!", message=AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"))], [ChatGeneration(text=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!", message=AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"))]], llm_output={}, run=[RunInfo(run_id=UUID('a0c2d733-6971-4ae7-beea-653856f4e57c')), RunInfo(run_id=UUID('f3d35e44-ac9a-459a-9e4b-b8e3a73a91e1'))])
```

```python
for chunk in chat.stream(messages):
    print(chunk.content, end="", flush=True)
```

```output
 Knock, knock!
Who's there?
Cows go.
Cows go who?
MOO!
```

您还可以使用所有异步 API 的功能：`ainvoke`、`abatch`、`agenerate` 和 `astream`。

```python
await chat.ainvoke(messages)
```

```output
AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!")
```

```python
await chat.abatch([messages, messages])
```

```output
[AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"),
 AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!")]
```

```python
await chat.agenerate([messages, messages])
```

```output
LLMResult(generations=[[ChatGeneration(text=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!", message=AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"))], [ChatGeneration(text=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!", message=AIMessage(content=" Knock, knock!\nWho's there?\nCows go.\nCows go who?\nMOO!"))]], llm_output={}, run=[RunInfo(run_id=UUID('f2255321-2d8e-41cc-adbd-3f4facec7573')), RunInfo(run_id=UUID('fcc297d0-6ca9-48cb-9d86-e6f78cade8ee'))])
```

```python
async for chunk in chat.astream(messages):
    print(chunk.content, end="", flush=True)
```

```output
 Knock, knock!
Who's there?
Cows go.
Cows go who?
MOO!
```