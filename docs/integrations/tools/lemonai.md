# Lemon Agent

[柠檬代理](https://github.com/felixbrock/lemon-agent) 可以帮助您在几分钟内构建强大的人工智能助手，并通过在诸如 `Airtable`, `Hubspot`, `Discord`, `Notion`, `Slack` 和 `Github` 等工具中进行准确可靠的读写操作来自动化工作流程。

查看[完整文档](https://github.com/felixbrock/lemonai-py-client)。

如今大多数可用的连接器都专注于只读操作，限制了 LLMs 的潜力。另一方面，代理有时会因缺少上下文或指令而产生幻觉。

通过 `Lemon AI`，您可以让您的代理访问定义良好的 API，进行可靠的读写操作。此外，`Lemon AI` 函数还允许您通过提供一种静态定义工作流程的方式来进一步减少幻觉的风险，模型在不确定情况下可以依赖这些工作流程。

## 快速开始

以下快速入门演示了如何将 Lemon AI 与代理结合使用，以自动化涉及内部工具交互的工作流程。

### 1. 安装 Lemon AI

需要 Python 3.8.1 及以上版本。

要在您的 Python 项目中使用 Lemon AI，请运行 `pip install lemonai`。

这将安装相应的 Lemon AI 客户端，然后您可以将其导入到您的脚本中。

该工具使用 Python 包 langchain 和 loguru。如果 Lemon AI 安装出现任何错误，请先安装这两个包，然后再安装 Lemon AI 包。

### 2. 启动服务器

您的代理与 Lemon AI 提供的所有工具的交互都由 [Lemon AI 服务器](https://github.com/felixbrock/lemonai-server) 处理。要使用 Lemon AI，您需要在本地计算机上运行服务器，以便 Lemon AI Python 客户端可以连接到它。

### 3. 使用 Lemon AI 与 Langchain

Lemon AI 通过找到相关工具的正确组合或使用 Lemon AI 函数来自动解决给定的任务。以下示例演示了如何从 Hackernews 检索用户并将其写入 Airtable 表格：

#### (可选) 定义您的 Lemon AI 函数

类似于 [OpenAI 函数](https://openai.com/blog/function-calling-and-other-api-updates)，Lemon AI 提供了定义工作流程作为可重用函数的选项。这些函数可以为那些尽可能接近确定性行为的用例定义，特定工作流程可以在单独的 lemonai.json 中定义：

```json
[
  {
    "name": "Hackernews Airtable User Workflow",
    "description": "retrieves user data from Hackernews and appends it to a table in Airtable",
    "tools": ["hackernews-get-user", "airtable-append-data"]
  }
]
```

您的模型将可以访问这些函数，并且会优先使用它们来解决给定的任务。您所需要做的就是让代理知道它应该使用给定的函数，方法是在提示中包含函数名称。

#### 在您的 Langchain 项目中包含 Lemon AI

```python
import os
from langchain_openai import OpenAI
from lemonai import execute_workflow
```

#### 加载 API 密钥和访问令牌

要使用需要身份验证的工具，您必须以格式 "{工具名称}_{身份验证字符串}" 将相应的访问凭据存储在您的环境中，其中身份验证字符串是 API 密钥的一种 ["API_KEY", "SECRET_KEY", "SUBSCRIPTION_KEY", "ACCESS_KEY"] 或身份验证令牌的一种 ["ACCESS_TOKEN", "SECRET_TOKEN"]。例如 "OPENAI_API_KEY", "BING_SUBSCRIPTION_KEY", "AIRTABLE_ACCESS_TOKEN"。

```python
""" 将所有相关的 API 密钥和访问令牌加载到您的环境变量中 """
os.environ["OPENAI_API_KEY"] = "*在此处插入 OpenAI API 密钥*"
os.environ["AIRTABLE_ACCESS_TOKEN"] = "*在此处插入 Airtable 令牌*"
```

```python
hackernews_username = "*在此处插入 Hackernews 用户名*"
airtable_base_id = "*在此处插入基地 ID*"
airtable_table_id = "*在此处插入表格 ID*"
""" 定义您要给您的 LLM 的指令 """
prompt = f"""从 Hackernews 读取用户 {hackernews_username} 的信息，然后将结果写入到
Airtable（baseId: {airtable_base_id}，tableId: {airtable_table_id}）。请确保 Airtable 不会自动转换字段类型。
"""
"""
使用 Lemon AI execute_workflow 包装器
以在 Lemon AI 的组合中运行您的 Langchain 代理
"""
model = OpenAI(temperature=0)
execute_workflow(llm=model, prompt_string=prompt)
```

### 4. 了解代理决策的透明度

为了了解您的代理如何与 Lemon AI 工具交互来解决给定的任务，所有做出的决定、使用的工具和执行的操作都将写入到本地的 `lemonai.log` 文件中。每当您的 LLM 代理与 Lemon AI 工具栈交互时，都会创建相应的日志条目。

```log
2023-06-26T11:50:27.708785+0100 - b5f91c59-8487-45c2-800a-156eac0c7dae - hackernews-get-user
2023-06-26T11:50:39.624035+0100 - b5f91c59-8487-45c2-800a-156eac0c7dae - airtable-append-data
2023-06-26T11:58:32.925228+0100 - 5efe603c-9898-4143-b99a-55b50007ed9d - hackernews-get-user
2023-06-26T11:58:43.988788+0100 - 5efe603c-9898-4143-b99a-55b50007ed9d - airtable-append-data
```

通过使用 [Lemon AI Analytics](https://github.com/felixbrock/lemon-agent/blob/main/apps/analytics/README.md)，您可以轻松地更好地了解工具的使用频率和顺序。因此，您可以确定代理程序决策能力的薄弱环节，并通过定义 Lemon AI 函数来实现更确定性的行为。