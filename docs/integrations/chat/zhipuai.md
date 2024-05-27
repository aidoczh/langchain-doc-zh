---

sidebar_label: 智谱 AI

---

# 智谱 AI

本笔记展示了如何在 LangChain 中使用 [智谱 AI API](https://open.bigmodel.cn/dev/api) ，并结合 langchain.chat_models.ChatZhipuAI 进行操作。

>[*GLM-4*](https://open.bigmodel.cn/) 是一个多语言大型语言模型，与人类意图保持一致，具有问答、多轮对话和代码生成等功能。新一代基础模型 GLM-4 的整体性能相比上一代有了显著提升，支持更长的上下文；更强大的多模态性；支持更快的推理速度，更高的并发性，大大降低推理成本；同时，GLM-4 增强了智能代理的能力。

## 入门指南

### 安装

首先，请确保 zhipuai 包已在您的 Python 环境中安装。运行以下命令：

```python
#!pip install --upgrade httpx httpx-sse PyJWT
```

### 导入所需模块

安装完成后，在您的 Python 脚本中导入必要的模块：

```python
from langchain_community.chat_models import ChatZhipuAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
```

### 设置 API 密钥

登录 [智谱 AI](https://open.bigmodel.cn/login?redirect=%2Fusercenter%2Fapikeys) 获取 API 密钥以访问我们的模型。

```python
import os
os.environ["ZHIPUAI_API_KEY"] = "zhipuai_api_key"
```

### 初始化智谱 AI 聊天模型

以下是初始化聊天模型的方法：

```python
chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
)
```

### 基本用法

像这样使用系统消息和用户消息调用模型：

```python
messages = [
    AIMessage(content="Hi."),
    SystemMessage(content="Your role is a poet."),
    HumanMessage(content="Write a short poem about AI in four lines."),
]
```

```python
response = chat.invoke(messages)
print(response.content)  # 显示 AI 生成的诗
```

## 高级功能

### 流支持

对于持续交互，请使用流功能：

```python
from langchain_core.callbacks.manager import CallbackManager
from langchain_core.callbacks.streaming_stdout import StreamingStdOutCallbackHandler
```

```python
streaming_chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
    streaming=True,
    callback_manager=CallbackManager([StreamingStdOutCallbackHandler()]),
)
```

```python
streaming_chat(messages)
```

### 异步调用

对于非阻塞调用，请使用异步方法：

```python
async_chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
)
```

```python
response = await async_chat.agenerate([messages])
print(response)
```

### 使用函数调用

GLM-4 模型也可以与函数调用一起使用，使用以下代码运行一个简单的 LangChain json_chat_agent。

```python
os.environ["TAVILY_API_KEY"] = "tavily_api_key"
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_json_chat_agent
from langchain_community.tools.tavily_search import TavilySearchResults
tools = [TavilySearchResults(max_results=1)]
prompt = hub.pull("hwchase17/react-chat-json")
llm = ChatZhipuAI(temperature=0.01, model="glm-4")
agent = create_json_chat_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)
```

```python
agent_executor.invoke({"input": "what is LangChain?"})
```