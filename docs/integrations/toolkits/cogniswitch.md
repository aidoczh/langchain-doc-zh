## Cogniswitch 工具

**使用 CogniSwitch 构建可无缝消费、组织和检索知识的生产就绪应用程序。使用您选择的框架，例如 Langchain，CogniSwitch 有助于减轻在选择正确的存储和检索格式时的决策压力。它还消除了在生成响应时的可靠性问题和幻觉。通过仅两个简单步骤与您的知识进行交互来开始。**

访问 [https://www.cogniswitch.ai/developer 进行注册](https://www.cogniswitch.ai/developer?utm_source=langchain&utm_medium=langchainbuild&utm_id=dev)。

**注册:**

- 使用您的电子邮件注册并验证您的注册

- 您将收到一封包含平台令牌和用于使用服务的 OAuth 令牌的邮件。

**步骤 1: 实例化工具包并获取工具:**

- 使用 Cogniswitch 令牌、OpenAI API 密钥和 OAuth 令牌实例化 cogniswitch 工具包并获取工具。

**步骤 2: 使用工具和 llm 实例化代理:**

- 使用 cogniswitch 工具列表和 llm 实例化代理执行器。

**步骤 3: CogniSwitch 存储工具:**

***CogniSwitch 知识源文件工具***

- 使用代理通过提供文件路径上传文件。(当前支持的格式为 .pdf, .docx, .doc, .txt, .html)

- 文件中的内容将由 CogniSwitch 处理并存储在您的知识存储中。

***CogniSwitch 知识源 URL 工具***

- 使用代理上传 URL。

- URL 中的内容将由 CogniSwitch 处理并存储在您的知识存储中。

**步骤 4: CogniSwitch 状态工具:**

- 使用代理了解已上传文档的状态。

- 您还可以在 CogniSwitch 控制台中检查文档处理的状态。

**步骤 5: CogniSwitch 答案工具:**

- 使用代理提出您的问题。

- 您将从您的知识中得到答案作为响应。

### 导入必要的库

```python
import warnings
warnings.filterwarnings("ignore")
import os
from langchain.agents.agent_toolkits import create_conversational_retrieval_agent
from langchain_community.agent_toolkits import CogniswitchToolkit
from langchain_openai import ChatOpenAI
```

### Cogniswitch 平台令牌、OAuth 令牌和 OpenAI API 密钥

```python
cs_token = "您的 CogniSwitch 令牌"
OAI_token = "您的 OpenAI API 令牌"
oauth_token = "您的 CogniSwitch 认证令牌"
os.environ["OPENAI_API_KEY"] = OAI_token
```

### 使用凭据实例化 Cogniswitch 工具包

```python
cogniswitch_toolkit = CogniswitchToolkit(
    cs_token=cs_token, OAI_token=OAI_token, apiKey=oauth_token
)
```

### 获取 Cogniswitch 工具列表

```python
tool_lst = cogniswitch_toolkit.get_tools()
```

### 实例化 llm

```python
llm = ChatOpenAI(
    temperature=0,
    openai_api_key=OAI_token,
    max_tokens=1500,
    model_name="gpt-3.5-turbo-0613",
)
```

### 创建代理执行器

```python
agent_executor = create_conversational_retrieval_agent(llm, tool_lst, verbose=False)
```

### 调用代理上传 URL

```python
response = agent_executor.invoke("upload this url https://cogniswitch.ai/developer")
print(response["output"])
```

```output
URL https://cogniswitch.ai/developer 已成功上传。文档的状态目前正在处理中。处理完成后，您将收到电子邮件通知。
```

### 调用代理上传文件

```python
response = agent_executor.invoke("upload this file example_file.txt")
print(response["output"])
```

```output
文件 example_file.txt 已成功上传。文档的状态目前正在处理中。处理完成后，您将收到电子邮件通知。
```

### 调用代理获取文档状态

```python
response = agent_executor.invoke("Tell me the status of this document example_file.txt")
print(response["output"])
```

```output
文档 example_file.txt 的状态如下:
- 创建时间: 2024-01-22T19:07:42.000+00:00
- 修改时间: 2024-01-22T19:07:42.000+00:00
- 文档条目 ID: 153
- 状态: 0 (处理中)
- 原始文件名: example_file.txt
- 保存的文件名: 1705950460069example_file29393011.txt
文档目前正在处理中。
```

### 调用代理查询并获取答案

```python
response = agent_executor.invoke("How can cogniswitch help develop GenAI applications?")
print(response["output"])
```

```output
CogniSwitch 可以通过以下几种方式帮助开发 GenAI 应用程序:
1. 知识提取: CogniSwitch 可以从各种来源（如文档、网站和数据库）中提取知识。它可以分析和存储这些来源的数据，使得更容易访问和利用这些信息用于 GenAI 应用程序。
2. 自然语言处理: CogniSwitch 具有先进的自然语言处理能力。它可以理解和解释人类语言，使得 GenAI 应用程序能够以更具对话和直观的方式与用户交互。
3. 情感分析: CogniSwitch 可以分析文本数据的情感，如客户评论或社交媒体帖子。这在开发能够理解和回应用户情绪和观点的 GenAI 应用程序中非常有用。
4. 知识库集成: CogniSwitch 可以与现有的知识库集成或创建新的知识库。这使得 GenAI 应用程序可以访问大量信息，并为用户查询提供准确和相关的响应。
5. 文档分析: CogniSwitch 可以分析文档并提取实体、关系和概念等关键信息。这在开发能够理解和处理大量文本数据的 GenAI 应用程序中非常有价值。
总的来说，CogniSwitch 提供了一系列基于人工智能的能力，通过实现知识提取、自然语言处理、情感分析、知识库集成和文档分析，可以增强开发 GenAI 应用程序的能力。
```