# Infobip

这篇笔记展示了如何使用 [Infobip](https://www.infobip.com/) 的 API 包装器来发送短信和电子邮件。

Infobip 提供了许多服务，但这篇笔记将专注于短信和电子邮件服务。您可以在[这里](https://www.infobip.com/docs/api)找到有关 API 和其他渠道的更多信息。

## 设置

要使用这个工具，您需要拥有一个 Infobip 账户。您可以创建一个[免费试用账户](https://www.infobip.com/docs/essentials/free-trial)。

`InfobipAPIWrapper` 使用命名参数，您可以提供凭据：

- `infobip_api_key` - 您可以在您的[开发者工具](https://portal.infobip.com/dev/api-keys)中找到的[API 密钥](https://www.infobip.com/docs/essentials/api-authentication#api-key-header)。

- `infobip_base_url` - Infobip API 的[基本 URL](https://www.infobip.com/docs/essentials/base-url)。您可以使用默认值 `https://api.infobip.com/`。

您还可以将 `infobip_api_key` 和 `infobip_base_url` 作为环境变量 `INFOBIP_API_KEY` 和 `INFOBIP_BASE_URL` 提供。

## 发送短信

```python
from langchain_community.utilities.infobip import InfobipAPIWrapper
infobip: InfobipAPIWrapper = InfobipAPIWrapper()
infobip.run(
    to="41793026727",
    text="Hello, World!",
    sender="Langchain",
    channel="sms",
)
```

## 发送电子邮件

```python
from langchain_community.utilities.infobip import InfobipAPIWrapper
infobip: InfobipAPIWrapper = InfobipAPIWrapper()
infobip.run(
    to="test@example.com",
    sender="test@example.com",
    subject="example",
    body="example",
    channel="email",
)
```

# 如何在代理内使用

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_community.utilities.infobip import InfobipAPIWrapper
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_core.tools import StructuredTool
from langchain_openai import ChatOpenAI
instructions = "You are a coding teacher. You are teaching a student how to code. The student asks you a question. You answer the question."
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
llm = ChatOpenAI(temperature=0)
class EmailInput(BaseModel):
    body: str = Field(description="Email body text")
    to: str = Field(description="Email address to send to. Example: email@example.com")
    sender: str = Field(
        description="Email address to send from, must be 'validemail@example.com'"
    )
    subject: str = Field(description="Email subject")
    channel: str = Field(description="Email channel, must be 'email'")
infobip_api_wrapper: InfobipAPIWrapper = InfobipAPIWrapper()
infobip_tool = StructuredTool.from_function(
    name="infobip_email",
    description="Send Email via Infobip. If you need to send email, use infobip_email",
    func=infobip_api_wrapper.run,
    args_schema=EmailInput,
)
tools = [infobip_tool]
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
agent_executor.invoke(
    {
        "input": "Hi, can you please send me an example of Python recursion to my email email@example.com"
    }
)
```

```bash
> 进入新的 AgentExecutor 链...
调用: 使用 `{'body': 'Hi,\n\nHere is a simple example of a recursive function in Python:\n\n```\ndef factorial(n):\n    if n == 1:\n        return 1\n    else:\n        return n * factorial(n-1)\n```\n\nThis function calculates the factorial of a number. The factorial of a number is the product of all positive integers less than or equal to that number. The function calls itself with a smaller argument until it reaches the base case where n equals 1.\n\nBest,\nCoding Teacher', 'to': 'email@example.com', 'sender': 'validemail@example.com', 'subject': 'Python Recursion Example', 'channel': 'email'}` 发送一个 Python 递归示例到您的电子邮件。
我已经向您的电子邮件发送了一个 Python 递归示例，请查看您的收件箱。
> 链结束。
```