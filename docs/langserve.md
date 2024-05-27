# 🦜️🏓 LangServe

[![发布说明](https://img.shields.io/github/release/langchain-ai/langserve)](https://github.com/langchain-ai/langserve/releases)

[![下载量](https://static.pepy.tech/badge/langserve/month)](https://pepy.tech/project/langserve)

[![未解决问题](https://img.shields.io/github/issues-raw/langchain-ai/langserve)](https://github.com/langchain-ai/langserve/issues)

[![](https://dcbadge.vercel.app/api/server/6adMQxSpJS?compact=true&style=flat)](https://discord.com/channels/1038097195422978059/1170024642245832774)

🚩 我们将发布 LangServe 的托管版本，以便一键部署 LangChain 应用程序。[在此注册](https://airtable.com/apppQ9p5XuujRl3wJ/shrABpHWdxry8Bacm) 加入等待列表。

## 概述

[LangServe](https://github.com/langchain-ai/langserve) 帮助开发者将 `LangChain` [可运行和链](https://python.langchain.com/docs/expression_language/) 部署为 REST API。

该库集成了 [FastAPI](https://fastapi.tiangolo.com/) 并使用 [pydantic](https://docs.pydantic.dev/latest/) 进行数据验证。

此外，它提供了一个客户端，可用于调用部署在服务器上的可运行对象。JavaScript 客户端可在 [LangChain.js](https://js.langchain.com/docs/ecosystem/langserve) 中找到。

## 特性

- 从 LangChain 对象自动推断输入和输出模式，并在每次 API 调用中执行，提供丰富的错误信息

- 带有 JSONSchema 和 Swagger 的 API 文档页面（插入示例链接）

- 高效的 `/invoke`、`/batch` 和 `/stream` 端点，支持单个服务器上的多个并发请求

- `/stream_log` 端点，用于流式传输链/代理的所有（或部分）中间步骤

- **新功能** 自 0.0.40 版本起，支持 `/stream_events`，使流式传输更加简便，无需解析 `/stream_log` 的输出。

- 在 `/playground/` 上提供内置的（可选的）跟踪到 [LangSmith](https://www.langchain.com/langsmith)，只需添加您的 API 密钥（参见[说明](https://docs.smith.langchain.com/)）

- 使用经过严格测试的开源 Python 库构建，如 FastAPI、Pydantic、uvloop 和 asyncio。

- 使用客户端 SDK 调用 LangServe 服务器，就像本地运行可运行对象一样（或直接调用 HTTP API）

- [LangServe Hub](https://github.com/langchain-ai/langchain/blob/master/templates/README.md)

## 限制

- 目前不支持服务器发起的事件的客户端回调

- 当使用 Pydantic V2 时，将不会生成 OpenAPI 文档。FastAPI 不支持[混合使用 pydantic v1 和 v2 命名空间](https://github.com/tiangolo/fastapi/issues/10360)。更多细节请参见下面的章节。

## 托管的 LangServe

我们将发布 LangServe 的托管版本，以便一键部署 LangChain 应用程序。[在此注册](https://airtable.com/apppQ9p5XuujRl3wJ/shrABpHWdxry8Bacm) 加入等待列表。

## 安全性

- 版本 0.0.13 - 0.0.15 中的漏洞 -- playground 端点允许访问服务器上的任意文件。[在 0.0.16 中解决](https://github.com/langchain-ai/langserve/pull/98)。

## 安装

对于客户端和服务器：

```bash
pip install "langserve[all]"
```

或者对于客户端代码，`pip install "langserve[client]"`，对于服务器代码，`pip install "langserve[server]"`。

## LangChain CLI 🛠️

使用 `LangChain` CLI 快速启动 `LangServe` 项目。

要使用 langchain CLI，请确保已安装最新版本的 `langchain-cli`。您可以使用 `pip install -U langchain-cli` 进行安装。

## 设置

**注意**：我们使用 `poetry` 进行依赖管理。请参阅 poetry [文档](https://python-poetry.org/docs/) 了解更多信息。

### 1. 使用 langchain cli 命令创建新应用

```sh
langchain app new my-app
```

### 2. 在 add_routes 中定义可运行对象。转到 server.py 并编辑

```sh
add_routes(app. NotImplemented)
```

### 3. 使用 `poetry` 添加第三方包（例如 langchain-openai、langchain-anthropic、langchain-mistral 等）

```sh
poetry add [package-name] // 例如 `poetry add langchain-openai`
```

### 4. 设置相关环境变量。例如，

```sh
export OPENAI_API_KEY="sk-..."
```

### 5. 启动您的应用

```sh
poetry run langchain serve --port=8100
```

## 示例

使用[LangChain 模板](https://github.com/langchain-ai/langchain/blob/master/templates/README.md) 快速启动您的 LangServe 实例。

有关更多示例，请参见模板[index](https://github.com/langchain-ai/langchain/blob/master/templates/docs/INDEX.md) 或 [examples](https://github.com/langchain-ai/langserve/tree/main/examples) 目录。

| 描述                                                                                                                                                                                                 | 链接                                                                                                                                                          |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LLMs** 最小示例，保留了 OpenAI 和 Anthropic 聊天模型。使用异步，支持批处理和流式处理。                                                                                                       | [server](https://github.com/langchain-ai/langserve/tree/main/examples/llm/server.py), [client](https://github.com/langchain-ai/langserve/blob/main/examples/llm/client.ipynb)  |
| **Retriever** 简单的服务器，将检索器公开为可运行的程序。                                                                                                                                       | [server](https://github.com/langchain-ai/langserve/tree/main/examples/retrieval/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/retrieval/client.ipynb)  |
| **Conversational Retriever** 通过 LangServe 公开的[对话检索器](https://python.langchain.com/docs/expression_language/cookbook/retrieval#conversational-retrieval-chain)                          | [server](https://github.com/langchain-ai/langserve/tree/main/examples/conversational_retrieval_chain/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/conversational_retrieval_chain/client.ipynb) |
| **Agent** 基于[OpenAI 工具](https://python.langchain.com/docs/modules/agents/agent_types/openai_functions_agent)的**无对话历史**代理                                                                 | [server](https://github.com/langchain-ai/langserve/tree/main/examples/agent/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/agent/client.ipynb)  |
| **Agent** 基于[OpenAI 工具](https://python.langchain.com/docs/modules/agents/agent_types/openai_functions_agent)的**有对话历史**代理                                                                 | [server](https://github.com/langchain-ai/langserve/blob/main/examples/agent_with_history/server.py), [client](https://github.com/langchain-ai/langserve/blob/main/examples/agent_with_history/client.ipynb)  |
| [带消息历史的可运行程序](https://python.langchain.com/docs/expression_language/how_to/message_history) 用于在后端实现持久化的聊天，由客户端提供的 `session_id` 作为键。                         | [server](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence/client.ipynb)  |
| [带消息历史的可运行程序](https://python.langchain.com/docs/expression_language/how_to/message_history) 用于在后端实现持久化的聊天，由客户端提供的 `conversation_id` 和 `user_id`（有关实现 `user_id` 的身份验证，请参见 Auth）作为键。 | [server](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence_and_user/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence_and_user/client.ipynb)  |
| [可配置的可运行程序](https://python.langchain.com/docs/expression_language/how_to/configure) 用于创建支持运行时配置索引名称的检索器。                                                                 | [server](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_retrieval/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_retrieval/client.ipynb)  |
| [可配置的可运行](https://python.langchain.com/docs/expression_language/how_to/configure) 展示了可配置字段和可配置的替代方案。                                                                                                      | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_chain/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_chain/client.ipynb)                         |
| **APIHandler** 展示了如何使用 `APIHandler` 而不是 `add_routes`。这为开发人员提供了更多灵活性来定义端点。与所有 FastAPI 模式兼容，但需要更多的工作。                                                        | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/server.py)                                                                                                                               |
| **LCEL 示例** 使用 LCEL 操作字典输入的示例。                                                                                                                                                                                          | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/passthrough_dict/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/passthrough_dict/client.ipynb)                             |
| 使用 `add_routes` 进行身份验证：可以应用于与应用相关的所有端点的简单身份验证。(单独使用时不适用于实现每个用户逻辑。)                                                                                           | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/global_deps/server.py)                                                                                                                                   |
| 使用 `add_routes` 进行身份验证：基于路径依赖的简单身份验证机制。(单独使用时不适用于实现每个用户逻辑。)                                                                                                                    | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/path_dependencies/server.py)                                                                                                                             |
| 使用 `add_routes` 进行身份验证：为使用每个请求配置修改器的端点实现每个用户逻辑和身份验证。(注意：目前不与 OpenAPI 文档集成。)                                                                                 | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/client.ipynb)     |
| 使用 `APIHandler` 进行身份验证：实现每个用户逻辑和身份验证，展示如何仅在用户拥有的文档中搜索。                                                                                                                                           | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/client.ipynb)                             |
| **小部件** 不同的小部件，可与 playground（文件上传和聊天）一起使用。                                                                                                                                                                              | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py)                                                                                                                                |
| **小部件** 用于 LangServe playground 的文件上传小部件。                                                                                                                                                                                                      | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/client.ipynb)                               |

## 示例应用

### 服务器

以下是一个部署 OpenAI 聊天模型、Anthropic 聊天模型以及使用 Anthropic 模型讲述有关特定主题笑话的链的服务器。

```python
#!/usr/bin/env python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatAnthropic, ChatOpenAI
from langserve import add_routes
app = FastAPI(
    title="LangChain 服务器",
    version="1.0",
    description="使用 Langchain 的 Runnable 接口的简单 API 服务器",
)
add_routes(
    app,
    ChatOpenAI(model="gpt-3.5-turbo-0125"),
    path="/openai",
)
add_routes(
    app,
    ChatAnthropic(model="claude-3-haiku-20240307"),
    path="/anthropic",
)
model = ChatAnthropic(model="claude-3-haiku-20240307")
prompt = ChatPromptTemplate.from_template("告诉我一个关于 {topic} 的笑话")
add_routes(
    app,
    prompt | model,
    path="/joke",
)
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

如果您打算从浏览器调用您的端点，您还需要设置 CORS 头。

您可以使用 FastAPI 的内置中间件来实现：

```python
from fastapi.middleware.cors import CORSMiddleware
# 设置所有启用 CORS 的来源
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["*"],
)
```

### 文档

如果您已部署上述服务器，可以使用以下命令查看生成的 OpenAPI 文档：

> ⚠️ 如果使用 pydantic v2，将不会为 _invoke_、_batch_、_stream_、_stream_log_ 生成文档。请参阅下面的 [Pydantic](#pydantic) 部分获取更多详细信息。

```sh
curl localhost:8000/docs
```

请确保**添加** `/docs` 后缀。

> ⚠️ 首页 `/` 没有被**设计**定义，因此 `curl localhost:8000` 或访问该 URL

> 将返回 404。如果您想在 `/` 上有内容，请定义一个端点 `@app.get("/")`。

### 客户端

Python SDK

```python
from langchain.schema import SystemMessage, HumanMessage
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnableMap
from langserve import RemoteRunnable
openai = RemoteRunnable("http://localhost:8000/openai/")
anthropic = RemoteRunnable("http://localhost:8000/anthropic/")
joke_chain = RemoteRunnable("http://localhost:8000/joke/")
joke_chain.invoke({"topic": "parrots"})
# 或者异步
await joke_chain.ainvoke({"topic": "parrots"})
prompt = [
    SystemMessage(content='Act like either a cat or a parrot.'),
    HumanMessage(content='Hello!')
]
# 支持 astream
async for msg in anthropic.astream(prompt):
    print(msg, end="", flush=True)
prompt = ChatPromptTemplate.from_messages(
    [("system", "Tell me a long story about {topic}")]
)
# 可以定义自定义链
chain = prompt | RunnableMap({
    "openai": openai,
    "anthropic": anthropic,
})
chain.batch([{"topic": "parrots"}, {"topic": "cats"}])
```

在 TypeScript 中（需要 LangChain.js 版本 0.0.166 或更高）：

```typescript
import { RemoteRunnable } from "@langchain/core/runnables/remote";
const chain = new RemoteRunnable({
  url: `http://localhost:8000/joke/`,
});
const result = await chain.invoke({
  topic: "cats",
});
```

使用 `requests` 的 Python 代码：

```python
import requests
response = requests.post(
    "http://localhost:8000/joke/invoke",
    json={'input': {'topic': 'cats'}}
)
response.json()
```

您也可以使用 `curl`：

```sh
curl --location --request POST 'http://localhost:8000/joke/invoke' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "input": {
            "topic": "cats"
        }
    }'
```

## 端点

以下代码：

```python
...
add_routes(
    app,
    runnable,
    path="/my_runnable",
)
```

将以下端点添加到服务器：

- `POST /my_runnable/invoke` - 对单个输入调用可运行项

- `POST /my_runnable/batch` - 对一批输入调用可运行项

- `POST /my_runnable/stream` - 对单个输入调用并流式传输输出

- `POST /my_runnable/stream_log` - 对单个输入调用并流式传输输出，

  包括生成的中间步骤的输出

- `POST /my_runnable/astream_events` - 对单个输入调用并在生成时流式传输事件，

  包括来自中间步骤的事件。

- `GET /my_runnable/input_schema` - 可运行项的输入的 JSON 模式

- `GET /my_runnable/output_schema` - 可运行项的输出的 JSON 模式

- `GET /my_runnable/config_schema` - 可运行项的配置的 JSON 模式

这些端点与

[LangChain 表达式语言接口](https://python.langchain.com/docs/expression_language/interface) 相匹配 --

请参考此文档以获取更多详细信息。

## Playground

您可以在 `/my_runnable/playground/` 找到一个可运行项的游乐场页面。这

提供了一个简单的 UI

来[配置](https://python.langchain.com/docs/expression_language/how_to/configure)

并调用您的可运行项，具有流式输出和中间步骤。

<p align="center">

<img src="https://github.com/langchain-ai/langserve/assets/3205522/5ca56e29-f1bb-40f4-84b5-15916384a276" width="50%"/>

</p>

### 小部件

游乐场支持[小部件](#playground-widgets)，可用于使用不同输入测试您的

可运行项。有关更多详细信息，请参阅下面的[小部件](#widgets)部分。

### 共享

此外，对于可配置的可运行项，游乐场将允许您配置

可运行项并共享带有配置的链接：

<p align="center">

<img src="https://github.com/langchain-ai/langserve/assets/3205522/86ce9c59-f8e4-4d08-9fa3-62030e0f521d" width="50%"/>

</p>

## 聊天游乐场

LangServe 还支持一个聊天重点的游乐场，可以选择并在 `/my_runnable/playground/` 下使用。

与一般游乐场不同，仅支持某些类型的可运行项 - 可运行项的输入模式必须为

一个 `dict`，其中包含以下内容之一：

- 一个键，该键的值必须是聊天消息列表。

- 两个键，一个键的值是消息列表，另一个代表最近的消息。

我们建议您使用第一种格式。

可运行项还必须返回 `AIMessage` 或字符串。

要启用它，必须在添加路由时设置 `playground_type="chat",`。以下是一个示例：

```python
# 声明一个对话链
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一个乐于助人、专业的助手，名叫 Cob。"),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
chain = prompt | ChatAnthropic(model="claude-2")
class InputChat(BaseModel):
    """聊天端点的输入。"""
    messages: List[Union[HumanMessage, AIMessage, SystemMessage]] = Field(
        ...,
        description="表示当前对话的聊天消息。",
    )
add_routes(
    app,
    chain.with_types(input_type=InputChat),
    enable_feedback_endpoint=True,
    enable_public_trace_link_endpoint=True,
    playground_type="chat",
)
```

如果您正在使用 LangSmith，您还可以在路由上设置 `enable_feedback_endpoint=True`，以在每条消息后面添加赞/踩按钮，

并设置 `enable_public_trace_link_endpoint=True`，以添加一个按钮，用于创建公共追踪记录。

请注意，您还需要设置以下环境变量：

```bash
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_PROJECT="YOUR_PROJECT_NAME"
export LANGCHAIN_API_KEY="YOUR_API_KEY"
```

以下是打开上述两个选项的示例：

<p align="center">

<img src="./.github/img/chat_playground.png" width="50%"/>

</p>

注意：如果启用了公共追踪链接，您的链的内部将被公开。我们建议仅在演示或测试中使用此设置。

## 旧版链

LangServe 可以与 Runnables（通过 [LangChain 表达语言](https://python.langchain.com/docs/expression_language/) 构建）

和旧版链（继承自 `Chain`）一起使用。

但是，旧版链的一些输入模式可能不完整/不正确，从而导致错误。

可以通过在 LangChain 中更新这些链的 `input_schema` 属性来修复此问题。

如果遇到任何错误，请在此存储库上提出问题，我们将努力解决。

## 部署

### 部署到 AWS

您可以使用 [AWS Copilot CLI](https://aws.github.io/copilot-cli/) 部署到 AWS

```bash
copilot init --app [application-name] --name [service-name] --type 'Load Balanced Web Service' --dockerfile './Dockerfile' --deploy
```

单击[此处](https://aws.amazon.com/containers/copilot/)了解更多信息。

### 部署到 Azure

您可以使用 Azure Container Apps（无服务器）部署到 Azure：

```
az containerapp up --name [container-app-name] --source . --resource-group [resource-group-name] --environment  [environment-name] --ingress external --target-port 8001 --env-vars=OPENAI_API_KEY=your_key
```

您可以在[此处](https://learn.microsoft.com/en-us/azure/container-apps/containerapp-up)找到更多信息。

### 部署到 GCP

您可以使用以下命令将其部署到 GCP Cloud Run：

```
gcloud run deploy [your-service-name] --source . --port 8001 --allow-unauthenticated --region us-central1 --set-env-vars=OPENAI_API_KEY=your_key
```

### 社区贡献

#### 部署到 Railway

[示例 Railway 存储库](https://github.com/PaulLockett/LangServe-Railway/tree/main)

[![在 Railway 上部署](https://railway.app/button.svg)](https://railway.app/template/pW9tXP?referralCode=c-aq4K)

## Pydantic

LangServe 对 Pydantic 2 提供支持，但有一些限制。

1. 使用 Pydantic V2 时，invoke/batch/stream/stream_log 的 OpenAPI 文档将不会生成。

   Fast API 不支持 [混合使用 pydantic v1 和 v2 命名空间]。

2. LangChain 在 Pydantic v2 中使用 v1 命名空间。请阅读

   [以下指南以确保与 LangChain 的兼容性](https://github.com/langchain-ai/langchain/discussions/9337)

除了这些限制外，我们预计 API 端点、playground 和其他任何功能都能正常工作。

## 高级

### 处理身份验证

如果需要为服务器添加身份验证，请阅读 Fast API 的文档

有关 [依赖项](https://fastapi.tiangolo.com/tutorial/dependencies/)

和 [安全性](https://fastapi.tiangolo.com/tutorial/security/)。

以下示例展示了如何使用 FastAPI 原语在 LangServe 端点中连接身份验证逻辑。

您需要提供实际的身份验证逻辑、用户表等。

如果不确定自己在做什么，可以尝试使用现有的解决方案 [Auth0](https://auth0.com/)。

#### 使用 add_routes

如果您使用 `add_routes`，请参阅

[此处的示例](https://github.com/langchain-ai/langserve/tree/main/examples/auth)。

| 描述                                                                                                                                                                               | 链接                                                                                                                                                                                                                           |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用 `add_routes` 进行身份验证：简单的身份验证机制，可应用于与应用程序相关的所有端点。 （单独使用时无法实现每个用户的逻辑。）           | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/global_deps/server.py)                                                                                                                               |
| 使用 `add_routes` 进行身份验证：基于路径依赖的简单身份验证机制。 （单独使用时无法实现每个用户的逻辑。）                                    | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/path_dependencies/server.py)                                                                                                                         |
| 使用 `add_routes` 进行身份验证：为使用每个请求配置修饰符的端点实现每个用户的逻辑和身份验证。 （注意：目前不与 OpenAPI 文档集成。） | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/client.ipynb) |

或者，您可以使用 FastAPI 的 [中间件](https://fastapi.tiangolo.com/tutorial/middleware/)。

使用全局依赖项和路径依赖项的优点是身份验证将在 OpenAPI 文档页面中得到正确支持，但是

这些不足以实现每个用户的逻辑（例如，创建一个只能在用户拥有的文档中搜索的应用程序）。

如果您需要实现每个用户的逻辑，可以使用 `per_req_config_modifier` 或 `APIHandler`（下面）来实现此逻辑。

**每个用户**

如果您需要与用户相关的授权或逻辑，

在使用 `add_routes` 时指定 `per_req_config_modifier`。使用一个可调用对象来接收

原始的 `Request` 对象，并从中提取相关信息进行身份验证和

授权目的。

#### 使用 APIHandler

如果您对 FastAPI 和 Python 感到熟悉，可以使用 LangServe 的 [APIHandler](https://github.com/langchain-ai/langserve/blob/main/examples/api_handler_examples/server.py)。

| 描述                                                                                                                                                                                                 | 链接                                                                                                                                                                                                           |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用 `APIHandler` 进行身份验证：实现每个用户的逻辑和身份验证，演示如何仅在用户拥有的文档中进行搜索。                                                                                    | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/client.ipynb)         |
| **APIHandler** 演示如何使用 `APIHandler` 而不是 `add_routes`。这为开发人员更灵活地定义端点提供了更多选择。与所有 FastAPI 模式配合使用效果很好，但需要更多的工作。 | [server](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/client.ipynb) |

这需要更多的工作，但可以完全控制端点定义，因此

您可以根据需要进行自定义身份验证逻辑。

### 文件

LLM 应用程序通常涉及文件处理。可以使用不同的架构

来实现文件处理；在高层次上：

1. 文件可以通过专用端点上传到服务器，并使用

   单独的端点进行处理

2. 文件可以通过值（文件的字节）或引用（例如，指向文件内容的 s3 URL）

   进行上传

3. 处理端点可以是阻塞或非阻塞的

4. 如果需要进行重要的处理，可以将处理工作转移到专用的

### 进程池

您应确定适合您的应用程序的适当架构。

目前，要通过值上传文件到可运行文件，请使用base64编码文件（`multipart/form-data`目前尚不支持）。

以下是一个[示例](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing)，展示如何使用base64编码将文件发送到远程可运行文件。

请记住，您始终可以通过引用（例如s3网址）上传文件，或将它们作为multipart/form-data上传到专用端点。

### 自定义输入和输出类型

所有可运行文件都定义了输入和输出类型。

您可以通过`input_schema`和`output_schema`属性访问它们。

`LangServe`使用这些类型进行验证和文档编制。

如果要覆盖默认推断类型，可以使用`with_types`方法。

以下是一个示例，用于说明这个概念：

```python
from typing import Any
from fastapi import FastAPI
from langchain.schema.runnable import RunnableLambda
app = FastAPI()
def func(x: Any) -> int:
    """应接受整数但接受任何类型的错误函数。"""
    return x + 1
runnable = RunnableLambda(func).with_types(
    input_type=int,
)
add_routes(app, runnable)
```

### 自定义用户类型

如果希望数据反序列化为pydantic模型而不是等效的字典表示，则继承自`CustomUserType`。

目前，此类型仅在服务器端起作用，并用于指定所需的解码行为。如果从此类型继承，服务器将保留解码类型作为pydantic模型，而不是将其转换为字典。

```python
from fastapi import FastAPI
from langchain.schema.runnable import RunnableLambda
from langserve import add_routes
from langserve.schema import CustomUserType
app = FastAPI()
class Foo(CustomUserType):
    bar: int
def func(foo: Foo) -> int:
    """期望一个Foo类型（pydantic模型）的示例函数。"""
    assert isinstance(foo, Foo)
    return foo.bar
add_routes(app, RunnableLambda(func), path="/foo")
```

### 游乐场小部件

游乐场允许您从后端为可运行文件定义自定义小部件。

以下是一些示例：

| 描述                                                                 | 链接                                                                 |
| :------------------------------------------------------------------ | --------------------------------------------------------------------- |
| **小部件** 不同的小部件，可与游乐场一起使用（文件上传和聊天） | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/client.ipynb) |
| **小部件** 用于LangServe游乐场的文件上传小部件。                   | [服务器端](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/server.py), [客户端](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/client.ipynb) |

#### 模式

- 小部件在字段级别指定，并作为输入类型的JSON模式的一部分进行传送

- 小部件必须包含一个名为`type`的键，其值是众所周知的小部件列表之一

- 其他小部件键将与描述JSON对象中路径的值相关联

```typescript
type JsonPath = number | string | (number | string)[];
type NameSpacedPath = { title: string; path: JsonPath }; // 使用title模拟json模式，但可以使用命名空间
type OneOfPath = { oneOf: JsonPath[] };
type Widget = {
  type: string; // 一些众所周知的类型（例如，base64file，chat等）
  [key: string]: JsonPath | NameSpacedPath | OneOfPath;
};
```

### 可用小部件

目前用户可以手动指定的小部件仅有两种：

1. 文件上传小部件

2. 聊天历史小部件

请参阅以下关于这些小部件的更多信息。

游乐场UI上的所有其他小部件都是根据可运行文件的配置模式自动创建和管理的。当您创建可配置的可运行文件时，游乐场应为您创建适当的小部件以控制行为。

#### 文件上传小部件

允许在UI游乐场中创建文件上传输入。

这些文件以 base64 编码字符串的形式上传。这里是完整的[示例](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing)。

代码片段：

```python
try:
    from pydantic.v1 import Field
except ImportError:
    from pydantic import Field
from langserve import CustomUserType
# 注意：继承自 CustomUserType 而不是 BaseModel，否则服务器会将其解码为字典而不是 pydantic 模型。
class FileProcessingRequest(CustomUserType):
    """包含 base64 编码文件的请求。"""
    # extra 字段用于为 playground UI 指定小部件。
    file: str = Field(..., extra={"widget": {"type": "base64file"}})
    num_chars: int = 100
```

示例小部件：

<p align="center">

<img src="https://github.com/langchain-ai/langserve/assets/3205522/52199e46-9464-4c2e-8be8-222250e08c3f" width="50%"/>

</p>

### 聊天小部件

查看[小部件示例](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py)。

要定义一个聊天小部件，请确保传递 "type": "chat"。

- "input" 是 _Request_ 中包含新输入消息的字段的 JSONPath。

- "output" 是 _Response_ 中包含新输出消息的字段的 JSONPath。

- 如果整个输入或输出应按原样使用（例如，如果输出是一系列聊天消息），则不要指定这些字段。

这里是一个代码片段：

```python
class ChatHistory(CustomUserType):
    chat_history: List[Tuple[str, str]] = Field(
        ...,
        examples=[[("human input", "ai response")]],
        extra={"widget": {"type": "chat", "input": "question", "output": "answer"}},
    )
    question: str
def _format_to_messages(input: ChatHistory) -> List[BaseMessage]:
    """将输入格式化为消息列表。"""
    history = input.chat_history
    user_input = input.question
    messages = []
    for human, ai in history:
        messages.append(HumanMessage(content=human))
        messages.append(AIMessage(content=ai))
    messages.append(HumanMessage(content=user_input))
    return messages
model = ChatOpenAI()
chat_model = RunnableParallel({"answer": (RunnableLambda(_format_to_messages) | model)})
add_routes(
    app,
    chat_model.with_types(input_type=ChatHistory),
    config_keys=["configurable"],
    path="/chat",
)
```

示例小部件：

<p align="center">

<img src="https://github.com/langchain-ai/langserve/assets/3205522/a71ff37b-a6a9-4857-a376-cf27c41d3ca4" width="50%"/>

</p>

您还可以直接将消息列表作为参数传递，如下所示：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assisstant named Cob."),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
chain = prompt | ChatAnthropic(model="claude-2")
class MessageListInput(BaseModel):
    """聊天端点的输入。"""
    messages: List[Union[HumanMessage, AIMessage]] = Field(
        ...,
        description="代表当前对话的聊天消息。",
        extra={"widget": {"type": "chat", "input": "messages"}},
    )
add_routes(
    app,
    chain.with_types(input_type=MessageListInput),
    path="/chat",
)
```

查看[此示例文件](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/message_list/server.py)以获取示例。

### 启用/禁用端点（LangServe >=0.0.33）

您可以在为给定链路添加路由时启用/禁用暴露的端点。

如果要确保在升级 langserve 到新版本时永远不会获得新端点，请使用 `enabled_endpoints`。

启用：下面的代码将仅启用 `invoke`、`batch` 和相应的 `config_hash` 端点变体。

```python
add_routes(app, chain, enabled_endpoints=["invoke", "batch", "config_hashes"], path="/mychain")
```

禁用：下面的代码将禁用链路的 playground。

```python
add_routes(app, chain, disabled_endpoints=["playground"], path="/mychain")
```