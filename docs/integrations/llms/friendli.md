---

sidebar_label: Friendli

---

# Friendli

> [Friendli](https://friendli.ai/) 通过可扩展、高效的部署选项增强AI应用程序性能，并针对高需求的AI工作负载进行优化，实现成本节约。

本教程将指导您如何将 `Friendli` 与 LangChain 集成。

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

您可以通过选择要使用的模型初始化 Friendli 聊天模型。默认模型是 `mixtral-8x7b-instruct-v0-1`。您可以在 [docs.friendli.ai](https://docs.periflow.ai/guides/serverless_endpoints/pricing#text-generation-models) 上查看可用模型。

```python
from langchain_community.llms.friendli import Friendli
llm = Friendli(model="mixtral-8x7b-instruct-v0-1", max_tokens=100, temperature=0)
```

## 用法

`Frienli` 支持所有 [`LLM`](/docs/how_to#llms) 的方法，包括异步API。

您可以使用 `invoke`、`batch`、`generate` 和 `stream` 的功能。

```python
llm.invoke("Tell me a joke.")
```

```output
'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"'
```

```python
llm.batch(["Tell me a joke.", "Tell me a joke."])
```

```output
['Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"',
 'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"']
```

```python
llm.generate(["Tell me a joke.", "Tell me a joke."])
```

```output
LLMResult(generations=[[Generation(text='Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"')], [Generation(text='Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"')]], llm_output={'model': 'mixtral-8x7b-instruct-v0-1'}, run=[RunInfo(run_id=UUID('a2009600-baae-4f5a-9f69-23b2bc916e4c')), RunInfo(run_id=UUID('acaf0838-242c-4255-85aa-8a62b675d046')])
```

```python
for chunk in llm.stream("Tell me a joke."):
    print(chunk, end="", flush=True)
```

```output
Username checks out.
User 1: I'm not sure if you're being sarcastic or not, but I'll take it as a compliment.
User 0: I'm not being sarcastic. I'm just saying that your username is very fitting.
User 1: Oh, I thought you were saying that I'm a "dumbass" because I'm a "dumbass" who "checks out"
```

您还可以使用所有异步API的功能：`ainvoke`、`abatch`、`agenerate` 和 `astream`。

```python
await llm.ainvoke("Tell me a joke.")
```

```output
'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"'
```

```python
await llm.abatch(["Tell me a joke.", "Tell me a joke."])
```

```output
['Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"',
 'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"']
```

```python
await llm.agenerate(["讲个笑话。", "讲个笑话。"])
```

```output
LLMResult(generations=[[Generation(text="用户名检查通过。\n用户1：我不确定你是认真的还是开玩笑，但我会把它当作是赞美。\n用户0：我是认真的。我不确定你是认真的还是开玩笑。\n用户1：我是认真的。我不确定你是认真的还是开玩笑。\n用户0：我是认真的。我不确定")], [Generation(text="用户名检查通过。\n用户1：我不确定你是认真的还是开玩笑，但我会把它当作是赞美。\n用户0：我是认真的。我不确定你是认真的还是开玩笑。\n用户1：我是认真的。我不确定你是认真的还是开玩笑。\n用户0：我是认真的。我不确定")]], llm_output={'model': 'mixtral-8x7b-instruct-v0-1'}, run=[RunInfo(run_id=UUID('46144905-7350-4531-a4db-22e6a827c6e3')), RunInfo(run_id=UUID('e2b06c30-ffff-48cf-b792-be91f2144aa6'))])
```

```python
async for chunk in llm.astream("讲个笑话。"):
    print(chunk, end="", flush=True)
```

```output
用户名检查通过。
用户1：我不确定你是在讽刺还是认真，但我会把它当作是赞美。
用户0：我不是在讽刺。我只是说你的用户名非常贴切。
用户1：哦，我以为你在说我是一个“蠢蛋”，因为我是一个“蠢蛋”而且“检查通过”。
```