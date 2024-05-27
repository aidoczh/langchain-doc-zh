---

sidebar_position: 0

---

# 构建一个简单的 LLM 应用程序

在这个快速入门中，我们将向您展示如何构建一个简单的 LLM 应用程序。这个应用程序将把英文文本翻译成其他语言。这是一个相对简单的 LLM 应用程序 - 它只是一个单独的 LLM 调用加上一些提示。尽管如此，这是一个很好的开始使用 LangChain 的方式 - 只需要一些提示和一个 LLM 调用，就可以构建许多功能！

## 概念

我们将涵盖的概念有：

- 使用[语言模型](/docs/concepts/#chat-models)

- 使用[PromptTemplates](/docs/concepts/#prompt-templates)和[OutputParsers](/docs/concepts/#output-parsers)

- 使用 LangChain 链接 PromptTemplate + LLM + OutputParser

- 使用[LangSmith](/docs/concepts/#langsmith)调试和跟踪您的应用程序

- 使用[LangServe](/docs/concepts/#langserve)部署您的应用程序

这是相当多的内容！让我们开始吧。

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用[Jupyter笔记本](https://jupyter.org/)，并假设读者也在使用。Jupyter笔记本非常适合学习如何使用LLM系统，因为往往会出现一些问题（意外的输出，API停机等），在交互式环境中阅读指南是更好地理解它们的好方法。

这个和其他教程最方便的运行方式可能是在Jupyter笔记本中。请参阅[这里](https://jupyter.org/install)以获取安装说明。

### 安装

要安装 LangChain，请运行以下命令：

import Tabs from '@theme/Tabs';

import TabItem from '@theme/TabItem';

import CodeBlock from "@theme/CodeBlock";

<Tabs>

  <TabItem value="pip" label="Pip" default>

    <CodeBlock language="bash">pip install langchain</CodeBlock>

  </TabItem>

  <TabItem value="conda" label="Conda">

    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>

  </TabItem>

</Tabs>

有关更多详细信息，请参阅我们的[安装指南](/docs/how_to/installation)。

### LangSmith

使用 LangChain 构建的许多应用程序将包含多个步骤，其中包含多个 LLM 调用。

随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。

最好的方法是使用[LangSmith](https://smith.langchain.com)。

在上面的链接上注册后，请确保设置您的环境变量以开始记录跟踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，如果在笔记本中，您可以使用以下代码设置它们：

```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 详细步骤

在本指南中，我们将构建一个应用程序，将用户输入从一种语言翻译为另一种语言。

## 使用语言模型

首先，让我们学习如何单独使用语言模型。LangChain 支持许多不同的语言模型，您可以互换使用 - 在下面选择要使用的模型！

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs openaiParams={`model="gpt-4"`} />

让我们首先直接使用模型。`ChatModel`是 LangChain "Runnables" 的实例，这意味着它们提供了与之交互的标准接口。要仅仅调用模型，我们可以将一系列消息传递给 `.invoke` 方法。

```python
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="Translate the following from English into Italian"),
    HumanMessage(content="hi!"),
]
model.invoke(messages)
```

```output
AIMessage(content='ciao!', response_metadata={'token_usage': {'completion_tokens': 3, 'prompt_tokens': 20, 'total_tokens': 23}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-fc5d7c88-9615-48ab-a3c7-425232b562c5-0')
```

如果我们启用了 LangSmith，我们可以看到此运行已记录到 LangSmith，并且可以查看[LangSmith跟踪](https://smith.langchain.com/public/88baa0b2-7c1a-4d09-ba30-a47985dde2ea/r)。

## OutputParsers

请注意，模型的响应是一个 `AIMessage`。它包含一个字符串响应以及有关响应的其他元数据。通常，我们可能只想使用字符串响应。我们可以使用一个简单的输出解析器来解析出这个响应。

首先，我们导入简单的输出解析器。

```python
from langchain_core.output_parsers import StrOutputParser
parser = StrOutputParser()
```

使用它的一种方式是单独使用它。例如，我们可以保存语言模型调用的结果，然后将其传递给解析器。

```python
result = model.invoke(messages)
parser.invoke(result)
```

```output
'Ciao!'
```

更常见的做法是，我们可以使用输出解析器来“链接”模型。这意味着在这个链条中，每次都会调用输出解析器。这个链条的输入类型是语言模型的输出（字符串或消息列表），输出类型是输出解析器的输出（字符串）。

我们可以使用 `|` 运算符轻松创建这个链条。`|` 运算符在 LangChain 中用于将两个元素组合在一起。

```python
chain = model | parser
```

```python
chain.invoke(messages)
```

```output
'Ciao!'
```

如果我们现在看一下 LangSmith，我们会发现这个链条有两个步骤：首先调用语言模型，然后将其结果传递给输出解析器。我们可以在 [LangSmith 跟踪](https://smith.langchain.com/public/f1bdf656-2739-42f7-ac7f-0f1dd712322f/r) 中看到这一点。

## 提示模板

现在，我们将直接将消息列表传递给语言模型。这个消息列表是从哪里来的呢？通常，它是由用户输入和应用逻辑的组合构成的。这个应用逻辑通常会将原始用户输入转换为一个准备传递给语言模型的消息列表。常见的转换包括添加系统消息或使用用户输入格式化模板。

PromptTemplates 是 LangChain 中设计用于辅助这种转换的概念。它们接收原始用户输入，并返回准备传递给语言模型的数据（提示）。

让我们在这里创建一个 PromptTemplate。它将接收两个用户变量：

- `language`：要将文本翻译成的语言

- `text`：要翻译的文本

```python
from langchain_core.prompts import ChatPromptTemplate
```

首先，让我们创建一个字符串，我们将对其进行格式化，以成为系统消息：

```python
system_template = "Translate the following into {language}:"
```

接下来，我们可以创建 PromptTemplate。这将是 `system_template` 的组合，以及一个更简单的模板，用于放置文本：

```python
prompt_template = ChatPromptTemplate.from_messages(
    [("system", system_template), ("user", "{text}")]
)
```

这个提示模板的输入是一个字典。我们可以单独玩弄这个提示模板，看看它单独做了什么：

```python
result = prompt_template.invoke({"language": "italian", "text": "hi"})
result
```

```output
ChatPromptValue(messages=[SystemMessage(content='Translate the following into italian:'), HumanMessage(content='hi')])
```

我们可以看到它返回了一个 `ChatPromptValue`，其中包含两条消息。如果我们想直接访问这些消息，可以这样做：

```python
result.to_messages()
```

```output
[SystemMessage(content='Translate the following into italian:'), HumanMessage(content='hi')]
```

现在，我们可以将其与上面的模型和输出解析器组合在一起。这将把所有三个组件链接在一起：

```python
chain = prompt_template | model | parser
```

```python
chain.invoke({"language": "italian", "text": "hi"})
```

```output
'ciao'
```

如果我们查看 LangSmith 跟踪，我们可以看到所有三个组件都出现在 [LangSmith 跟踪](https://smith.langchain.com/public/bc49bec0-6b13-4726-967f-dbd3448b786d/r) 中。

## 使用 LangServe 进行服务

现在我们已经构建了一个应用程序，我们需要对其进行服务。这就是 LangServe 的用武之地。

LangServe 帮助开发人员将 LangChain 链作为 REST API 进行部署。您不需要使用 LangServe 来使用 LangChain，但在本指南中，我们将展示如何使用 LangServe 来部署您的应用程序。

虽然本指南的第一部分是在 Jupyter Notebook 或脚本中运行的，但我们现在将离开那个环境。我们将创建一个 Python 文件，然后从命令行与其交互。

使用以下命令安装：

```bash
pip install "langserve[all]"
```

### 服务器

要为我们的应用程序创建一个服务器，我们将创建一个 `serve.py` 文件。这将包含我们用于提供应用程序的逻辑。它包括三个部分：

1. 我们刚刚构建的链的定义

2. 我们的 FastAPI 应用程序

3. 用于提供链的路由的定义，使用 `langserve.add_routes` 完成

```python
#!/usr/bin/env python
from typing import List
from fastapi import FastAPI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langserve import add_routes
# 1. 创建提示模板
system_template = "Translate the following into {language}:"
prompt_template = ChatPromptTemplate.from_messages([
    ('system', system_template),
    ('user', '{text}')
])
# 2. 创建模型
model = ChatOpenAI()
# 3. 创建解析器
parser = StrOutputParser()
# 4. 创建链
chain = prompt_template | model | parser
# 4. 应用程序定义
app = FastAPI(
  title="LangChain Server",
  version="1.0",
  description="A simple API server using LangChain's Runnable interfaces",
)
# 5. 添加链路由
add_routes(
    app,
    chain,
    path="/chain",
)
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

这就是全部内容！如果我们执行这个文件：

```bash
python serve.py
```

我们应该在 [http://localhost:8000](http://localhost:8000) 上看到我们的链正在提供服务。

### 游乐场

每个 LangServe 服务都配有一个简单的[内置用户界面](https://github.com/langchain-ai/langserve/blob/main/README.md#playground)，用于配置和调用具有流式输出和中间步骤可见性的应用程序。

前往 [http://localhost:8000/chain/playground/](http://localhost:8000/chain/playground/) 进行尝试！输入与之前相同的参数 - `{"language": "italian", "text": "hi"}` - 它应该会像之前一样回应。

### 客户端

现在让我们为与我们的服务进行程序化交互设置一个客户端。我们可以使用 `[langserve.RemoteRunnable](/docs/langserve/#client)` 轻松实现这一点。

使用它，我们可以像在客户端运行一样与提供的链进行交互。

```python
from langserve import RemoteRunnable
remote_chain = RemoteRunnable("http://localhost:8000/chain/")
remote_chain.invoke({"language": "italian", "text": "hi"})
```

```output
'Ciao'
```

要了解更多关于 LangServe 的其他功能，请[点击这里](/docs/langserve)。

## 结论

就是这样！在本教程中，我们已经介绍了如何创建我们的第一个简单的 LLM 应用程序。我们学会了如何使用语言模型，如何解析它们的输出，如何创建提示模板，如何在使用 LangSmith 创建的链中获得良好的可观察性，以及如何使用 LangServe 部署它们。

这只是你想要学习成为一名熟练的 AI 工程师的表面。幸运的是 - 我们还有很多其他资源！

要了解更详细的教程，请查看我们的[Tutorials](/docs/tutorials)部分。

如果您对如何完成特定任务有具体问题，请查看我们的[How-To Guides](/docs/how_to)部分。

要了解 LangChain 的核心概念，请阅读我们详细的[Conceptual Guides](/docs/concepts)。